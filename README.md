# 모던 자바 인 액션 스터디

스터디 구성원과 조정 예정인 문서입니다.

## 목표

- 모던 자바 인 액션에 나오는 모든 챕터들을 함께 공부한다.
- 이해되지 않는 문장이나 개념이 있다면, 공부하여 공유한다.
- 이 스터디를 통해 스트림과 더불어 리액티브 프로그래밍과 함수형 프로그래밍을 이해한다.
- 공부한 것을 해당 repository 에 문서로 남긴다.

## 기간

- 챕터 21개 X 1주 = 총 21주
- 2023년 3월 4일 ~ 2023년 7월 22일
- 매주 토요일 (8:00 ~ 1시간 or 2시간) 화상으로 진행
- 첫번째 모임 시작일 2023.03.04 (토)

## 스터디 방식

챕터 단위로 진행
20챕터

기본적으로 책은 모두 읽는다가 전제
1챕터에 있는 소주제들을 나누어서 담당 -> 담당한다는 의미는 문서작성, 질문에 대한 답변 책임감 부여 가장 먼저 답변을 해야할 사람 (물론 모른다면 토론 혹은 좀더 조사)
담당자 대표 문서 취합


1. 스터디 진행 방식은 매주 미팅를 통해 조금씩 개선하는 방식을 택한다.
    - 예시 : 주간 미팅 -> 챕터 공유 -> 스터디 방식에 대한 의견 조율
    - 일주일간 진행하면서, 아쉬웠던 부분 혹은 어떻게 개선해야 겠다는 생각, 그 주 자신과 관련된 이슈 등을 말하도록 한다.

2. 매주 챕터 1개씩, 챕터 내에 소주제에 대한 담장자 존재
    - 소주제들을 참여인원이 돌아가면서 담당한다.
    - 해당 주제 자체 내용이 적거나, 혹은 너무 어렵다면 할당에 대해 스터디원간 합의하여 유동적으로 조율한다

3. 매주 챕터에 대한 이슈 생성
    - 이슈 생성 시, 순서에 맞춰 생성한다.
    - Assignees 를 통해 챕터 담당자를 구별한다
    - 생성된 issue는 팀원들 간에 소통의 공간이다.
    - ⭐️팀원들은, 한 주간 정한 챕터에서 생겼던 질문들을 적합한 issue에 남기고, 특정 소주제에 대한 것이라면 그 소주제의 담당자가 대답을 남긴다.
        - 대답은, 간단한 줄글 부터 예시 코드까지 작성할 수 있음.
        - 최대한 답변자가 이해할 수 있게 쉽게 설명.
    - 담당자가 해결하지 못하는 질문 또는 내용은 매주 **토요일 미팅** 시간에 공유하여, 해결하도록 한다.
       - 해결되지 못한 질문은 넘어가자 --> 너무 완벽하게, 깊게 파고들려고 하지 말자.
       - 해당 질문이 간단해서, 검색을 통해 해결될 수 있는 문제라면 조사 후, **issue 댓글로 정리 내용을 공유**하기로 한다.
    - 담당자가 책임감을 가지고 답변을 달되 자신의 담당이 아닌 댓글에도 모든 구성원들이 의견을 달 수 있다.
 

### 깃 컨벤션

- 담당자는 item에 대한 정리**(질답 내용을 포함)** 하여 markdown file로 commit 한다.
    - 커밋 메세지는, `[챕터-소주제번호] 제목` 과 같이 챕터와 소주제와 함께 제목을 같이 작성한다 
        - 메세지 예시 --> `[3-1] 람다란 무엇인가?`
    - 문서 계층 구조 예시
        - modern-java-in-action (root)
            - chapter03
                - README.md -> 총 취합 문서
                - item01.md -> 소주제를 `item` 이라고 하자.
    - 해당 commit은 가능한 그 주내에 할 수 있도록 한다.
    - README에 item 정리한다 (스터디 끝나고 진행할 것)
    - branch는 모두 main으로 진행
        - 커밋 title, 은 issue 이름과 동일하게 한다.

- item 정리 방식
    - 개개인 담당자에게 맡기지만, 남들이 봤을 때도(초급자가 봤을 때도) 이해할 수 있을 정도로 정리.

- 위 스터디 방식은 팀원과의 합의를 통해, 언제든지 수정 가능하다.

## Ground Rule

- 예치금 30,000원
- 벌금
    - 매주 토요일 스터디 미참가 : 5000원
        - 누구나 참작가능하다고 생각하는 이유면 가능함.(당일 제외) 
            - ex) 경조사, 질병, 약속...
    - 담당부분에 대한 어떤 형태로든 응답이 전혀 없을시 : 5000원
- 진행자 역할은 @kyupid
    - 챕터 issue 만들기 
    - 문서 취합
    - 미팅 진행자

## 대화 수칙

- 우린 '근거'를 통해 피드백하며 '무분별한 비난'은 하지 않습니다. 
- 우린 '경청'을 노력하며, '피드백'을 환영합니다. 
- 우린 누구나 자유롭게 의견을 낼 수 있지만 '나만 정답이다'라는 태도를 지양합니다. 
