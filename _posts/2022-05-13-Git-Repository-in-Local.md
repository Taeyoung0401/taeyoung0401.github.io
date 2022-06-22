---
layout: post
title: 로컬 환경에서 Git Repository 사용하기
subtitle: 
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [git]
comments: true
---

Git repository를 로컬로 카피(Clone)하고 활용하는 방법에 대해 설명한다.


# Clone from Remote Server

```
git clone --mirror <original.git>
```

`--mirror` 옵션을 통해 git repository의 전체 정보를 클론한다.  
`--bare` 와의 차이점은 각자 검색해 보길 바란다.


# Clone from Local Repository

상기 `--mirror` 옵션을 통해 클론 받은 repository에서 다시 로컬로 클론할 수 있다.

```
git clone ./localrepo/your-project.git
```

클론 받은 디렉토리에서 아래 명령어를 통해 remote의 branch를 확인하고,  
Checkout하면 된다.

```
git branch -r
```

옵션이 없는 경우, 로컬의 브랜치만 표시 (현재는 master만 보임)  
`-r` 옵션은 리모트의 브랜치만  
`-a` 옵션은 로컬과 리모트 모두 표시

```
git checkout -t origin/remote-branch-name
```

`-t` 옵션에 대한 설명은 아래

```
-t
--track
  When creating a new branch, set up "upstream" configuration. See "--track" in git-branch(1) for details.
```

# Push to Local Repository

> 여기서부터는 Git를 사용하는 일반적인 내용임

로컬의 git에 브랜치도 생성하고 하면서 여러 커밋을 한 후, 이를 다시 push하는 것은 아래와 같은 방식을 이용하면 된다.

```
git push -u origin <my-branch>
```

상기 `-u` 옵션에 의해 리모트에도 브랜치가 생성되었다면, 그 다음의 push는 그냥 아래와 같이 가능함

```
git push
```

# Push to Remote Repository

로컬 repository에서 작업을 마친 후, 다시 리모트의 git에 해당 내용을 머지하고 싶은 경우 리모트 설정을 추가한 후 push하면 된다.

```
git remote add origin2 <original-remote.git>
git push -u origin2 <my-branch>
```
