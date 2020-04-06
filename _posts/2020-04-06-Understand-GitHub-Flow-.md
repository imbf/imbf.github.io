---
layout: post
title:  "GitHub Flow란?"
date:   2020-04-06
author: Green Frog Developer
categories: Git
tags: Git Github Flow Version
---

이 글은 GitHub Flow가 무엇인지에 대해 알기위해 [Github공식문서](https://guides.github.com/introduction/flow/)를 번역한 글이다.

### GitHub flow 이해하기

GitHub flow는 브랜치 기반의 가벼운 workflow 입니다. GitHub flow는 배포가 정기적으로 이루어지는 팀과 프로젝트를 지원합니다. 

지금부터 GitHub flow를 어떻게 사용하는지에 대해서 설명 하겠습니다.

---

## 1. Create a branch

---

<img src="/assets/Git/GitHub-Flow-1.png" style="width:90%">


프로젝트를 진행간에 사용자는 특정 시점에 진행중인 다양한 기능 또는 다양한 아이디어의 brnach를 가지고 있습니다. 그 중 일부는 준비가 되어 있기도 하고 아직 준비가 안된것이 있기도 합니다. 또한 branch를 나누는것은 사용자가 이러한 workflow를 관리하는데 도움을 줍니다.

사용자가 본인의 프로젝트에서 branch를 생성할 때, 사용자는 새로운 아이디어를 도전할 환경을 생성합니다. 이러한 브랜치를 만드는 변화는 `master` branch에 영향을 주지 않기 때문에, 사용자는 자유롭게 실험하고, 공동 작업중인 누군가가 검토할 준비가 될 때까지 브랜치가 병합되지 않는다는 사실을 알고 변경 사항을 commit합니다.

### ProTip

Branch를 나누는것은 Git의 핵심 개념이고 전체 GitHub flow는 Branching에 기초합니다. 한가지의 가장 중요한 규칙은 **`master` branch에 있는 모든 것은 항상 배포할 수 있어야 한다는 것입니다.**

이러한 규칙 때문에, 기능 또는 수정 작업시 새로운 branch는 마스터에서 생성하는 것이 매우 중요합니다. 다른 작업자들이 어떠한 작업들이 진행되는지 알 수 있도록 branch 이름은 **descriptive**(설명적이어야 한다 ex. refactor-authentication, user-content-cache-key, make-retina-avatars) 이어야 한다.

---

## 2. Add commits

---

<img src="/assets/Git/GitHub-Flow-2.png" style="width:90%">


branch가 생성이 되어지면, 변화를 시작해야 한다. 사용자는 파일을 추가, 삭제, 편집할 때마다 commit을 하고 brach에 변경사항을 추가합니다. 이러한 commit을 추가하는 과정은 기능 branch에서 작업 진행 상황을 추적합니다.

또한 Commit은 다른 사람이 수행한 작업과 이유를 이해하기 위해 따라갈 수 있는 투명한 작업 기록을 만듭니다. 각가의 Commit은 관련된 특정한 변화가 왜 만들어 졌는지에 대해 설명하는 Commit message를 가지고 있습니다. 게다가 각각의 Commit은 분리된 변화의 단위로써 고려되어집니다. 이러한 것들은 버그가 발견되었을 경우에 뒤로 후퇴할 수 있고, 다른 방향으로 머리를 돌릴 수 있도록 해줍니다.

### ProTip

Commit Messages는 Git이 변화를 추적하고 서버로 push될 때 그들을 보여주기 때문에 특히 중요하다. Commit Message를 명확하게 작성함으로써, 개발자는 다른 사람들이 쉽게 따라하고 피드백을 제공할 수 있게 해줍니다.

---

## 3. Open a Pull Request

---

<img src="/assets/Git/GitHub-Flow-3.png" style="width:90%">


Pull Reqeusts는 Commits에 대한 토론을 시작합니다. Git Repository 기반에서 철처하게 통합이 되기 때문에, 요청을 수락하면 누구나 어떤 변경 내용이 병합되는지 정확하게 볼 수 있다.

개발 프로세스동안에 어느 지점에서도 Pull Request를 열 수 있다. ex) 코드가 적거나 없어도 몇개의 스크린샷 또는 일반적인 아이디어를 팀원들과 공유하고 싶을때, 팀원들의 도움이나 조언이 필요할 때 또는 누군가가 작업을 검토 할 준비가 되었을 때

Pull Request message에서 GitHub의 @mention 시스템을 사용함으로써, 개발자는 특정한 사람 또는 팀으로 시간대에 상관없이 피드백을 요청할 수 있습니다.

### ProTip

Pull Request는 공유된 저장소에 변화를 관리하기 위해, 오픈 소스 프로젝트에 기여하기 위해 매우 유용하다. 만약 Fork & Pull Model을 사용한다면 Pull Request는 그것들에게 기여한 변화에 대해서 Project maintainers에게 알려줄 방법을 제공할 것이다. 만약 공유되어지는 Repository Model을 사용하고 있다면, Pull Requests는 master brnach에 머지되기전에 제안된 변화들에 대해서 대화와 코드리뷰를 할 수 있도록 도와준다.

---

## 4. Discuss and review your code

---

<img src="/assets/Git/GitHub-Flow-4.png" style="width:90%">


Pull Request가 열려지면, 너의 변화를 리뷰하는 사람 또는 팀은 아마 질문과 comments를 가지고 있을 것이다. 아마도 코딩 스타일이 프로젝트 지침에 따르지 않았거나, 변경 사항에 단위 테스트가 누락되었거나 모든 것이 멋지게 보이고 구성요소들이 순서대로 표시되었을 수 있습니다. Pull Requests는 이러한 유형의 대화를 장려하고 캡처하도록 설계되었습니다.

또한 커밋에 대한 토론과 피드백을 반영해 브랜치에 계쏙 푸시할 수도 있습니다. 만약 누군가가 당신에게 어떠한 것을 잊어버리셨다 또는 코드에 버그가 있다고 comments를 남기면, 당신은 본인의 브랜치에서 이러한 피드백들을 고치고 변화를 push할 수 있습니다. GitHub는 통합 Pull Requests View에서 새로운 Comiit과 다른 추가적인 피드백들을 보여줄 것이다. 

### ProTip

Pull Requests Comments는 마크다운에서 쓰여지며, 개발자는 이미지와 이모지, 다른 전처리되는 text blocks들을 사용해 삽입할 수 있습니다.

---

## 5. Deploy

---

<img src="/assets/Git/GitHub-Flow-5.png" style="width:90%">


GitHub와 함께, 개발자는 branch로부터 마스터에 merge하기전에 구현체에 대해서 최종 테스트를 위해 배포할 수 있습니다.

Pull Request요청이 검토되어지고, branch가 tests를 통과하면, 개발자는 프로덕션 환경에서 이러한 것들을 검증하기 위해 변화를 배포할 수 있습니다. 만약 branch가 이슈를 생성하면, 기존 마스터를 프로덕션에 배포하여 지점을 후퇴할 수 있습니다.

---

## 6. Merge

---

<img src="/assets/Git/GitHub-Flow-6.png" style="width:90%">


변화들이 프로덕션 환경에서 검증이 되어지면, 코드를 master branch로 통합할 때가 온것입니다.

한번 merge가 되어지면, Pull Requests는 코드 변화의 기록을 보존하고, 누구든지 왜 어떻게 이러한 결정이 만들어졌는지 이해하기 위해 직접 볼 수 있습니다.

### ProTip

Pull Requests 키워드의 텍스트에 특정 키워드를 통합하면 이슈를 코드와 연결할 수 있습니다. Pull Request이 merge되면 관련된 이슈들도 또한 닫힙니다.


---
> 참조 
> - https://guides.github.com/introduction/flow/
