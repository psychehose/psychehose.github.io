## UMG에서 오디오 트랙을 정보를 얻는 방법


UMG 사운드 출력은 애니메이션에서 오디오 트랙을 추가해서 할 것이다.
이때, UMG에서 사운드가 출력될 때 코드 처리가 필요하거나 로깅을 해야할 수 있다.

UMG에서 오디오 트랙 정보를 얻고, 이 애니메이션이 재생될 때 브로드캐스팅을 받는 방법이다.

* {Project}.Build.cs에 "MovieScene", "MovieSceneTracks" 의존성 추가해야함.


```cpp

// Dependency header
#include "Tracks/MovieSceneAudioTrack.h"
#include "Sections/MovieSceneAudioSection.h"



void UUCustomWidget::OnAnimationStarted_Implementation(const UWidgetAnimation* Animation)
{
	Super::OnAnimationStarted_Implementation(Animation);

#if defined(WITH_EDITOR) && defined(__FUNC_LOG_DETAIL_SOUND__)
	U2_LOG(U2Sound, Log, TEXT("Animation started: %s"), *Animation->GetName());
	GetAudioTrackInfo(Animation);
#endif
}

void UUCustomWidget::GetAudioTrackInfo(const UWidgetAnimation* Animation) const
{
	if (!Animation)
		return;
	if (!Animation->GetMovieScene())
		return;

	UMovieScene* MovieScene = Animation->GetMovieScene();

	TArray<UMovieSceneTrack*> AudioTracks = MovieScene->GetMasterTracks();

	for (UMovieSceneTrack* Track : AudioTracks)
	{
		if (UMovieSceneAudioTrack* AudioTrack = Cast<UMovieSceneAudioTrack>(Track))
		{
			// Get Audio Tack info

			TArray<UMovieSceneSection*> Sections = AudioTrack->GetAllSections();
			for (UMovieSceneSection* Section : Sections)
			{
				// Process Section
				if (UMovieSceneAudioSection* AudioSection = Cast<UMovieSceneAudioSection>(Section))
				{
					if (AudioSection->HasStartFrame())
					{
						FFrameNumber StartFrame = AudioSection->GetInclusiveStartFrame();
						float StartTime = MovieScene->GetTickResolution().AsSeconds(StartFrame);

						U2_LOG(U2Sound, Log, TEXT("@@@@@@@@@@@@@@@@@@@@@@Audio Track Start Time: %f @@@@@@@@@@@@@@@@@@@"), StartTime);

						USoundBase* Sound = AudioSection->GetSound();
						if (Sound)
						{
							FString SoundName = Sound->GetName();

							FTimerHandle LogSoundTimer;

							// #TDBH - will later implement LogSound in Soundmanager

							//GetWorld()->GetTimerManager().SetTimer(LogSoundTimer, FTimerDelegate::CreateLambda([SoundName]()
							//	{
							//		UU2SoundManager::Instance()->LogSound(SoundName);
							//	}), StartTime, false);
						}
					}
				}
			}
		}
	}
}

```