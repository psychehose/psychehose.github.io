
```bash
$ git rm -r --cached {path}
$ git commit -m "deleted" # 현재 해시에서 삭제함.
$ git filter-branch --force --index-filter \
"git rm -r --cached --ignore-unmatch {path}" \
--prune-empty --tag-name-filter cat -- --all # 리모트 저장소 히스토리에서 삭제
```

