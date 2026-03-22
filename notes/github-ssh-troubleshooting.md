# GitHub SSH & 인증 트러블슈팅


---

## 1. Permission denied (publickey)

**에러 메시지**
```
git@github.com: Permission denied (publickey).
```

**원인**  
SSH config가 없거나, 공개키가 GitHub에 등록되지 않은 상태

**해결**  
1. `~/.ssh/config` 파일 생성 및 작성
```
Host github.com-miri
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_miri
```
2. GitHub → Settings → SSH and GPG keys → 공개키 등록 (`.pub` 파일 내용)
3. `ssh -T git@github.com-miri` 로 테스트

---

## 2. 계정 충돌 (403 에러)

**에러 메시지**
```
remote: Permission to mirikim404/repo.git denied to mill2kko.
fatal: unable to access '...': The requested URL returned error: 403
```

**원인**  
Git이 다른 계정(mill2kko)의 인증 정보를 캐시해서 계속 그걸로 push 시도

**해결**  
1. Windows 자격 증명 관리자에서 GitHub 관련 항목 전부 삭제
2. remote URL을 SSH 방식으로 변경
```bash
git remote set-url origin git@github.com-miri:mirikim404/레포이름.git
```
3. 다시 push → 로그인 창 뜨면 올바른 계정으로 로그인

> 💡 HTTPS 방식은 계정 캐시 충돌이 잦음. SSH 방식이 훨씬 안정적

---

## 3. SSH config 문법 오류

**에러 메시지**
```
/c/Users/.../.ssh/config line 4: keyword identityfile extra arguments at end of line
```

**원인**  
config 파일의 `IdentityFile` 줄에 오타 또는 불필요한 문자 포함

**해결**  
`nano ~/.ssh/config` 열어서 아래처럼 정확히 수정
```
Host github.com-miri
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_miri
```
- `=` 쓰면 안 됨
- 줄 끝에 공백/추가 문자 없어야 함
- `.pub` 붙이면 안 됨 (비공개키 경로 써야 함)

---

## 4. Could not open a connection to your authentication agent

**에러 메시지**
```
Could not open a connection to your authentication agent.
```

**원인**  
`ssh-add` 실행 전에 `ssh-agent`가 켜져 있지 않은 상태

**해결**  
```bash
eval "$(ssh-agent -s)"       # agent 켜기
ssh-add ~/.ssh/id_ed25519_miri   # 키 등록
```

> 💡 Git Bash는 ssh-agent가 자동 실행 안 될 때가 있어서 수동으로 켜야 함

---

## 5. SSH 키가 이미 사용 중 (Key is already in use)

**에러 메시지**
```
Key is already in use
```

**원인**  
같은 SSH 키가 다른 계정(mill2kko)에 이미 등록되어 있음

**해결**  
새 키를 다른 이름으로 생성
```bash
ssh-keygen -t ed25519 -C "mirikim404" -f ~/.ssh/id_ed25519_miri
```
그 다음 새 공개키를 올바른 계정에 등록

---

## 6. [rejected] fetch first

**에러 메시지**
```
! [rejected] main -> main (fetch first)
```

**원인**  
GitHub 레포에 이미 파일이 있는데 (README 체크 등), 로컬은 그걸 모르고 push 시도

**해결**  
```bash
git pull origin main --allow-unrelated-histories
git push -u origin main
```

---

## 7. fatal: not a git repository

**에러 메시지**
```
fatal: not a git repository (or any of the parent directories): .git
```

**원인**  
현재 위치가 프로젝트 폴더가 아님 (홈 디렉토리 등에서 명령어 실행)

**해결**  
```bash
cd /c/loc-git/do-it-github-5days   # 실제 레포 폴더로 이동
git status                          # 제대로 들어왔는지 확인
```

---

## 8. 기타 꿀팁

| 상황 | 명령어 |
|------|--------|
| git log에서 나가기 | `q` |
| Windows에서 touch 안 될 때 | `nano 파일명` 또는 탐색기에서 직접 생성 |
| SSH 연결 테스트 | `ssh -T git@github.com-miri` |
| 현재 remote 확인 | `git remote -v` |
| 계정 확인 | `git config --global user.name` |
