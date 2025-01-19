
리모트 저장소에서 히스토리 지우기

```bash
$ git rm -r --cached {path}
$ git commit -m "deleted" # 현재 해시에서 삭제함.
$ git filter-branch --force --index-filter \
"git rm -r --cached --ignore-unmatch {path}" \
--prune-empty --tag-name-filter cat -- --all # 리모트 저장소 히스토리에서 삭제
```


reset 커밋 복구

```bash
git reflog
git reset --hard 복구하려는커밋해시
```

- `git reset --hard`  현재 작업 중인 변경사항을 모두 삭제하므로 주의하기
* 작업 중인 변경사항을 보존:   `--soft` 사용하기