
Swarm에  커스텀 모듈을 만들어야 한다.


```tree
$SWARM_ROOT

config
└──custom.modules.config.php

module
└── Teams
    ├── Module.php
    ├── config
    │   └── module.config.php
    └── src
        └── Listener
            └── TeamsActivityListener.php
```


* **custom.modules.config.php**
	*  클래스를 자동 로드하고 어떤 사용자 정의 모듈을 실행해야 하는지 확인
	* 


- **module.config.php**
    - Swarm이 모듈을 인식하도록 하는 설정 파일
    - PHP 배열 형태로 이벤트 리스너 바인딩, 서비스 등록

      
- **Module.php**
    - **module.config.php**를 불러오는 파일.
    - namespace 설정해야함
    - getConfig()를 구현해야함


```php
// custom.modules.config.php
<?php
\Laminas\Loader\AutoloaderFactory::factory(
    array(
        'Laminas\Loader\StandardAutoloader' => array(
            'namespaces' => array(
                'Teams'      => BASE_PATH . '/module/Teams/src',
            )
        )
    )
);
return [
    'Teams'
];
```

```php
// Module.php
<?php
/**
 * Perforce Swarm Teams Module
 *
 * @author     psyche95
 */

namespace Teams;

class Module
{
    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php';
    }
}
```


```php

// module.config.php
<?php
/**
 * Perforce Swarm Teams Module
 *
 * @author     psyche95
 */

$listeners = [Teams\Listener\TeamsActivityListener::class];

return [
    'listeners' => $listeners,
    'service_manager' =>[
        'factories' => array_fill_keys(
            $listeners,
            Events\Listener\ListenerFactory::class
        )
    ],
    Events\Listener\ListenerFactory::EVENT_LISTENER_CONFIG => [
        Events\Listener\ListenerFactory::ALL => [
            Teams\Listener\TeamsActivityListener::class => [
                [
                    Events\Listener\ListenerFactory::PRIORITY => Events\Listener\ListenerFactory::HANDLE_MAIL_PRIORITY + 1,
                    Events\Listener\ListenerFactory::CALLBACK => 'handleEvent', // callback 함수 등록
                    Events\Listener\ListenerFactory::MANAGER_CONTEXT => 'queue' // custom module 사용
                ]
            ]
        ],
    ],
    'teams' => [
        'swarm_host' => '',
        'teams_webhook' => '',
        'email' => '',
        'id' => '',
    ]
];
```

* `Events\Listener\ListenerFactory::CALLBACK => 'handleEvent'` : callback 함수 등록

* `Events\Listener\ListenerFactory::MANAGER_CONTEXT => 'queue'`: custom module 사용

* `'teams' => []`  : src에서 멤버 변수로 사용할 hashmap



`custom.module.config.php`에서 지정한 `BASE_PATH`에 로직을 작성할 소스파일을 생성하고 구현한다.

```php

// src/Listener/TeamsActivityLister.php
<?php
/**
 * Perforce Swarm Teams Module
 *
 * @author     psyche95
*/

namespace Teams\Listener;

use Events\Listener\AbstractEventListener;
use Reviews\Model\Review;
use Comments\Model\Comment;
use Laminas\EventManager\Event;
use Laminas\Http\Client;
use Laminas\Http\Request;

class TeamsActivityListener extends AbstractEventListener
{

    public function handleEvent(Event $event)
    {
        $activity = $event->getParam('activity');
        if (!$activity)
        {
            return;
        }
        $logger = $this->services->get('logger');
        $config = $this->services->get('config');
        $p4Admin = $this->services->get('p4_admin');
        // Host address of Swarm for link back URLs
        $host = $this->services->get('ViewHelperManager')->get('qualifiedUrl');

        $logger->info("Teams: event / id => " . $event->getName() . " / " . $event->getParam('id'));
        
		// taken from mail module, this doesn't seem to work
        $data  = (array) $event->getParam('data') + array('quiet' => null);
        $quiet = $event->getParam('quiet', $data['quiet']);
        if ($quiet === true) {
            $logger->info("Teams: event is silent(notifications are being batched), returning.");
            return;
        }
        
		// it's better not to tag user involved in activity, only review author
		//$user = $this->tagUser($config, $activity->get('user'));
        $user = $activity->get('user');
        $action = $activity->get('action');
        $target = $activity->get('target');
        $link = $activity->get('link');
		$projects = $activity->get('projects');

		$logger->err("Projects => " . json_encode($projects));

        if (count($link) > 0)
        {
            if (count($link) > 1)
            {
                $link = $host($link[0], $link[1]);
            }
            else
            {
                $link = $host($link[0]);
            }
        }

		$eventString = $user . " " . $action . " " . $target;
		$linkMessage = $link;
		$linkMessage = str_replace('localhost', $config['teams']['swarm_host'], $linkMessage);
	
		$reviewId = -1;
		switch($event->getName())
		{
			case "task.comment.batch":
				if (preg_match('/^reviews\/(\d+)$/', $event->getParam('id'), $matches))
				{
					$reviewId = $matches[1];
					$eventString = "Comments have been made on review " . $reviewId . " => " . $host('review', array('review' => $reviewId));
				}
				break;
			case "task.review":
				$reviewId = $event->getParam('id');
				$eventData = $event->getParam('data');
				if (isset($eventData['isAdd']) && $eventData['isAdd'])
				{
					//new review
					$reviewId = 0;
				}
				$logger->info("Teams: event data => " . json_encode($eventData));
				break;
			case "task.comment":
				try {
					$comment = Comment::fetch($event->getParam('id'), $p4Admin);
					$topic = $comment->get('topic');
					if (preg_match('/^reviews\/(\d+)$/', $topic, $matches))
					{
						$reviewId = $matches[1];
					}
				} catch (\Exception $e) {
					$logger->err("Teams: error when fetching comment : " . $e->getMessage());
				}
				//don't treat single comments, wait for the comment.batch
				return;
				break;
			default:
				$logger->info("Teams: event not treated " . $eventString);
				return;
		}
		
		$notify = "";
		if ($reviewId > 0)
		{
			try {
				$review = Review::fetch($reviewId, $p4Admin);
				$reviewAuthor = $review->get('author');
			} catch (\Exception $e) {
				$logger->err("Teams: error when fetching review : " . $e->getMessage());
			}
		}
		else if ($reviewId == 0)
		{
			//notify everybody
			$notify = '@everyone ';
		}
		
		$eventString = "Hey " . $notify . "! " . $eventString;
        $logger->info("Teams: " . $eventString);

		$leadid = "";
		$leadname = "";
		$projname = "";

		if (!empty($projects)) {
			foreach ($projects as $key => $projectArray) {
				switch ($key) {
					case "":
						break 2;  
					case "":
						break 2;  
				}
			}
		}

		// URL to POST messages to Teams
		$teamsUrl = $config['teams']['teams_webhook'];
        $this->postTeams($teamsUrl, $eventString, $leadid, $leadname, $projname, $linkMessage);
		
        $logger->info("Teams: handleEvent end.");
    }

    private function postTeams($url, $msg, $leadid, $leadname, $projname, $linkMessage)
    {
        $logger = $this->services->get('logger');
        $config = $this->services->get('config');

		$team_leader_id = $config['teams'][''];
		$team_leader_name = $config['teams'][''];

        try {

			$messageData = [
				'type' => 'message',
				'attachments' => [
					[
						'contentType' => 'application/vnd.microsoft.card.adaptive',
						'content' => [
							'width' => 'Full',
							'type' => 'AdaptiveCard',
							'body' => [
								[
									'type' => 'TextBlock',
									'size' => 'Medium',
									'weight' => 'Bolder',
									'text' => $projname . ' Perforce Swarm Notification'
								],
								[
									'type' => 'TextBlock',
									'text' => '<at>' . $team_leader_name . '</at> <at>' . $leadname . '</at>'
								],
								[
									'type' => 'TextBlock',
									'text' => $msg
								],
								[
									'type' => 'ActionSet',
									'actions' => [
										[
											'type' => 'Action.OpenUrl',
											'title' => 'Go to Swarm',
											'url' => $linkMessage
										]
									]
								]
							],
							'$schema' => 'http://adaptivecards.io/schemas/adaptive-card.json',
							'version' => '1.0',
							'msteams' => [
								'entities' => [
									[
										'type' => 'mention',
										'text' => '<at>' . $team_leader_name . '</at>',
										'mentioned' => [
											'id' =>  $team_leader_id,
											'name' => $team_leader_name
										]
									],
									[
										'type' => 'mention',
										'text' => '<at>' . $leadname . '</at>',
										'mentioned' => [
											'id' =>  $leadid,
											'name' => $leadname
										]
									]
								]
							]
						]
					]
				]
			];

            $client = new Client();
            $client->setUri($url);
            $client->setMethod(Request::METHOD_POST);
            $client->setRawBody(json_encode($messageData));
            $client->setHeaders(['Content-Type' => 'application/json']);
            
            $response = $client->send();
            if (!$response->isSuccess()) {
                $logger->err("Teams: Failed to send message. Status: " . $response->getStatusCode());
            }
        } catch (\Exception $e) {
            $logger->err("Teams: Error sending message: " . $e->getMessage());
        }
    }
}
```


