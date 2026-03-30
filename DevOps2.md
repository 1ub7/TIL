# GitHub 명령어 정리
### 1. 저장소(Repository) 관련
프로젝트 시작 / 연결  
```
git init 
- 현재 폴더를 Git 저장소로 만듦
- .git 폴더 생성됨   
```
```
git clone
- 원격 저장소를 내 컴퓨터로 그대로 복사
```
```
git remote add origin 주소
- 원격 저장소 연결
- GitHub 연결할 때 필수
```
```
git remote -v
- 원격 확인
```
```
git remote set-url origin 주소
- 원격 변경
```
### 2. 파일 상태/확인
현재 상황 파악
```
git status
- 수정된 파일
- 추가 안 된 파일
- 커밋 상태
```
```
git log
- 커밋 기록 확인
```
```
git log --oneline
- 한 줄 보기
```
### 3.스테이징(Staging)
커밋 전에 올리는 단계
```
git add (파일명)
- 특정 파일 추가
- 파일명 빼면 전부 추가
```
```
git reset 파일명
- 스테이징 취소
```
### 4. 커밋(Commit)
변경사항 저장
```
git commit -m "메시지"
- 변경사항 기록
- 협업의 핵심이 됨
```
```
git commit -am "메시지"
- 수정된 파일만 가능
- add + commit
```
### 5. 브랜치(Branch)
작업 분리
```
git branch (이름)
- 브랜치 확인 및 생성
```
```
git checkout (이름)
- 브랜치 이동
```
```
git checkout -b (이름)
- 브랜치 생성 + 이동
```
### 6. 병합(Merge)
브랜치 합치기
```
git merge 브랜치명
- 다른 브랜치 내용 합침
- 충돌(conflict) 발생 가능
```
### 7. 원격 저장소
```
push - 업로드   
pull - 가져오기 + 병합   
fetch - 가져오기만
```
```
git push origin main - 업로드
git push -u origin main - 최초 연결 push
git pull origin main - 가져오기 / 자동 merge 포함
```
### 8. 되돌리기
```
git reset 파일명 - add 취소
git reset --soft HEAD~1 - 커밋 취소(유지)
git reset --hard HEAD~1 - 커밋 완전 취소(복구 어려움)
git checkout --파일명 - 수정 내용 되돌림 / 파일 복구
```
### 9. 임시 저장(Stash)
```
git stash - 임시 저장
git stash pop - 다시 꺼냄
```
### 10. 기타
```
git clone 주소 - GitHub 프로젝트 전체 다운로드 / 클론 복사
git diff - 변경된 내용 차이 (커밋 안 한 상태에서 확인 가능)
git show - 특정 커밋의 상세 내용 보기
```