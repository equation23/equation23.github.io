

# Hi-Fi Rush 모작 팀 포트폴리오 프로젝트
 **프로젝트 기간** 
 : 2024.09 ~ 2024.11
 
  **프로젝트 인원** 
  : 7명
  
 **담당 파트**
 : (팀장) 플레이어, Mimosa 보스 담당
-  [포트폴리오 팀 영상](https://youtu.be/rwTkUgP4ITA)
-  [담당파트 세부설명 영상](https://www.youtube.com/watch?v=s1WKke9D8nY) 

## 세부 사항
<details>
<summary>접기/펼치기</summary>

---------
# 목차
1. 개발내역
   1. 플레이어
      - 회피 상태
      - 점프 및 공중상태
      - 애니메이션
      - 콤보 공격
      - 피격/타격
   2. 어시스트   
   3. PhysX
      - CCT(Character Controller)
      - PhysX Visual Debuger
      - Raycast, Sweep
   4. 카메라
      - 메인 카메라
      - 이벤트 카메라
        
3. 트러블슈팅
   1. BGM 애니메이션 동기화
   2. 공격 애니메이션 to Idle 모션
---------
# 1. 개발 내역

### 1-1. 플레이어

- **이동 방식**
  
  - 플레이어의 이동 방식은 입력한 방향키에 따라 카메라를 기준으로한 방향벡터를 사용하여 8방향으로 이동할 수 있게 구현했습니다.

  - 현재 이동중인 방향과 입력된 방향이 다를 경우에는 플레이어의 현재 Look 벡터와 입력된 Look 벡터를 내적하여 각도를 구한 후,  
    외적하여 양수가 나올경우 시계방향 회전, 음수가 나올 경우 반시계 방향으로 회전 방향을 설정합니다.   
    회전 방향이 결정되면 Y축 회전을 0.1초 안에 시행하여 플레이어 모델이 회전하게끔 구현했습니다.

- **회피 상태**
  
   - 회피 버튼을 통해 연속으로 최대 3회까지 회피 애니메이션이 재생되며 회피 상태에 진입합니다.   
     회피 상태에서는 몬스터의 공격에 피격당하지 않게 피격시 예외처리하였습니다.

   - 회피 시에도 이동과 같이 8방향으로 회피를 구현하였습니다.   
     현재 회피중인 방향과 입력된 회피 방향이 다를경우 내적한 각도를 통하여 회전시키고, 해당 방향에 맞는 특수 회피 동작을 하도록 구현했습니다.

- **점프 및 공중상태**

  - 플레이어는 지상 상태일 경우 중력의 영향을 받지 않으나 공중에 있을 경우 중력의 영향을 받게 구현했습니다.   
    객체별 중력 가속도 및 중력 가속의 한계를 설정할 수 있습니다.

  - 점프 키 입력시 중력을 해제하고 플레이어의 Y축 방향으로 애니메이션 진행동안 매 프레임 감소하는 속도를 부여하여 점프를 구현했습니다
 
  - 점프 애니메이션이 끝난 경우 공중 상태에 진입하고 중력을 다시 적용합니다.
 
  - 공중 상태에서 지형과 플레이어의 하단 충돌이 일어날 경우 착지 상태로 진입하며 중력을 해제하고 중력 가속도를 초기화합니다.

  - 점프 및 공중 상태에서 이동시에는 기존의 이동과 다른 공중이동 상태를 적용해 이동하는 속도를 낮게 조정하였습니다.


- **애니메이션**

  - Hi-Fi Rush 게임에서 플레이어의 애니메이션은 항상 BGM의 BPM에 연동되어 동작합니다.   
    해당 기능을 구현하기 위해서 애니메이션 개별마다
    종료되는 BPM 길이를 설정하여 어떤 BGM이 배정되어도 항상 같은 BPM에 동작이 끝나게 구현했습니다.

  - 애니메이션의 선형보간을 적용했습니다.   
    공격 애니메이션들의 보간을 하는 과정은 BPM에 연동되어 동작하게 구현하기 위해, 공격 모션마다 다음 공격으로 넘어가는 KeyFrame을 지정하여 해당하는 KeyFrame에서
    다음 공격 애니메이션으로 넘어가게 구현했습니다.

- **콤보공격**
  
   - 플레이어의 공격은 좌클릭, 우클릭으로 시전할 수 있습니다.   
     콤보 공격은 좌클릭, 우클릭, 한박자 쉬기를 조합하여 시전할 수 있게 구현했습니다.

   - 콤보 분기를 최적화하기 위해 좌클릭, 우클릭, 한박자 쉬기로 이루어진 3진 Tree 구조로 구현했습니다.   
     플레이어 공격 입력시, 입력된 키의 Tree를 탐색하고 탐색한 노드의 값이 유효할 경우 해당 콤보 상태로 진입하게 구현했습니다.

   - 공격 상태중 공격 입력시에는 BPM과 애니메이션을 동기화 하기 위해, 입력된 콤보를 저장한 후 일정 KeyFrame에서 재생하게 하여
     공격 애니메이션을 BPM에 맞고 끊기지 않게 구현했습니다.
     
- **피격/타격**

  -  Collider Component를 활용하여 피격/타격에 대한 Collider를 원하는 객체에게 적용할 수 있도록 구현했습니다.
  -  Collider는 AABB, OBB, Sphere 의 3가지 종류를 구현했습니다.

  - 타격 Collider와 피격 Collider가 충돌 시 타격방향, 데미지, 피격 레벨의 정보를 피격 대상에게 전달하여 피격상태로 전환하게 구현했습니다.
    
- **필살기**
  
  - 필살기는 좌상단 UI와 연동하여 일정 수치의 에너지가 있을 때만 사용할 수 있게 구현했습니다.
    
  - 필살기 시전 시 카메라는 플레이어 모델의 카메라 Bone에 부착시켜 역동적인 카메라 움직임을 보이게 구현했습니다.


### 1-2. 어시스트
  
   - **어시스트 소환**
     - 어시스트 버튼을 입력시 좌상단 UI의 현재 어시스트가 소환되도록 구현했습니다.

     - 어시스트 애니메이션은 등장, 공격, 퇴장의 3단계로 나누어 등장시엔 플레이어의 Look 방향으로 소환되며   
       공격시에는 가장 가까운 적을 향해 회전합니다. 퇴장시에는 현재 카메라쪽으로 어시스트의 Look을 회전하여 퇴장하도록 구현했습니다.

     - 어시스트들 또한 BGM에 동기화된 애니메이션 재생을 구현하였습니다.

     - 소환 후에는 어시스트 개별 쿨타임이 적용되어 일정 시간이 지난 후 소환 가능하게 구현했습니다.
     
     - 어시스트와 협공하는 필살기 사용 시에는 어시스트를 플레이어의 중점에 소환 후   
       플레이어의 Look벡터를 어시스트에게 동일하게 적용하여 같은 대상에 협공하게 구현했습니다.

     
### 1-3. PhysX

- **CCT(Character Controller)**
  
  - CCT를 사용한 캡슐형 PhsyX Collider를 바탕으로 플레이어와 몬스터의 상호충돌 및 지형충돌을 구현했습니다.   
  - CCT기능을 필요한 객체에게 부여 할 수 있도록 Component화 하여 충돌이 필요한 객체들에게 적용할 수 있게끔 구현했습니다.   
  - 충돌식별이 필요한 객체들의 경우 PhysX Actor Name을 설정하여, 충돌시 Name 을 반환받아 객체별 처리를 용이하게 구현했습니다.   
  
- **PhysX Visual Debuger**
  
  - 효과적인 PhsyX 디버깅을 하기 위해 PhysX Visual Debuger를 사용한 실시간 충돌 정보를 시각적으로 확인하며 디버깅했습니다.


- **Raycast, Sweep**
  
  - 객체의 충돌 상태를 확인하기 위해 PhsyX Raycast, Sweep을 사용해 충돌 여부를 반환받아 충돌 상태처리를 구현했습니다.
  
  - 플레이어가 아주 조금만이라도 공중에 떠 있을경우에도 공중 상태에 진입하는것을 막기 위해 플레이어의 -Y축을 Direction 으로 설정한 Raycast를 통해
  일정한 거리 이내일 경우에는 공중 상태에 진입하지 않게 처리했습니다.
  
  - 투사체들이 벽 또는 다른 객체에 충돌여부를 반환받기 위해 Sweep을 사용했습니다, 투사체의 이동 방향을 Direction으로 설정하고 매 프레임 현재 속도를 입력하여
  투사체의 충돌 여부를 반환받고, 충돌시에는 충돌한 대상에 따른 개별 처리를 구현했습니다.

### 1-4. 카메라

- **메인 카메라**
  
  - 메인 카메라는 플레이어의 중점에서 일정한 거리를 두고 움직이게 구현했습니다.
  - 카메라는 마우스의 움직임에 따라 Quaternion 회전을 하도록 구현했습니다.
  - 필살기 사용시 카메라는 플레이어 모델의 카메라 Bone에 부착되어 움직임을 수행한 후 종료시에는   
    시전 전 위치로 보간되어 복귀합니다.
    
- **이벤트 카메라**

  - 특정 이벤트 상황에서 카메라에게 특정한 객체를 기준으로 방향, 거리를 부여하여 카메라를 고정 할 수 있게 구현했습니다.
  - 카메라 고정 시 입력한 시간동안 보간되어 움직입니다.
 

------------
# 2. 트러블 슈팅

  ### 2-1. BGM 애니메이션 동기화

  - **문제점**
    - 1.. BGM의 BPM에 맞게 애니메이션 동작을 맞춰야하기 떄문에 BPM을 초(sec) 단위로 변환하여 Beat라는 단위로 변환하였습니다.
   
    - 2.. 변환한 Beat에 맞게 애니메이션들의 동작을 정렬하였습니다. 대시는 4Beat의 재생속도를 가지고 일반공격들은 1~10사이의 Beat의 재생속도를
      가지게 구현하였습니다.

    - 3.. 하지만 Beat의 시작 부분에서 정확히 입력을 하지 않을 시 Beat에 맞게 애니메이션이 끝나지 않아 박자를 맞추지 못했습니다.
   
    - 4.. 반Beat 늦게 애니메이션을 실행 시에는 반 Beat 가 밀려 반박자가 밀려 리듬감이 구현되지않았습니다.

    - 5.. 정박자에 애니메이션이 실행하지 않아도 항상 Beat에 맞게 끝나는 애니메이션 동작을 구현해야했습니다.


 - **해결법**
    - 1.. 현재 Beat을 1/3으로 나눈 후, 애니메이션이 실행되는 순간 현재 Beat의 위치에 따라 재생속도를 조절했습니다.
  
    - 2.. 애니메이션 실행 명령이 들어왔을 때, 현재 Beat가 1/3 이하일 시에는 다음 Beat에 도달할때 까지 애니메이션 재생속도를   
      증가시켜 Beat에 맞게 동작하게 구현했습니다.

    - 3.. 반대로 Beat가 2/3 이상 구간부턴 재생속도를 증가시킬 시 애니메이션이 어색하게 빨라졌기 때문에 다른 방식으로 처리하였습니다.
      
    - 4.. Beat가 2/3 구간 이상일 경우엔  Beat + (Beat - 현재Beat시간)만큼의 재생시간을 더하여 총 애니메이션 길이는 1Beat가 늘지만 Beat의 끝나는 타이밍에 애니메이션이
      종료되게끔 구현하여 문제를 해결했습니다.

  ### 2-2. 공격 애니메이션 to Idle 모션

  - **문제점**
    - 1.. 공격 애니메이션이 끝날 시 Idle 모션으로 전환되는 동작이 포함되어있습니다.
   
    - 2.. 콤보 공격중 Idle 모션으로 전환되는 동작에 공격 입력 시, 공격 모션이 자연스럽게 이어지지 않는 문제가 발생하였습니다.
      
  - **해결법**
     - 1.. 공격 모션별 다음 공격 애니메이션으로 이어지는 부분을 확인하여 저장했습니다.
   
     - 2.. 상기 표시한 지점 이전에 공격 입력이 들어 올 경우 공격 입력을 저장한 후, 표시한 지점에서부터 재생하게 구현했습니다.
   
     - 3.. 상기 표시한 지점 이후에 공격 입력이 들어 올 경우에는 입력이 들어와도 Idle 모션이 나오게 구현했습니다.

</details>

-------------------------------

# ULTRAKILL 모작 포트폴리오 프로젝트
 **프로젝트 기간** 
 : 2024.06 ~ 2024.8

  **프로젝트 인원** 
 : 1명
 
-  [개인 포트폴리오 영상](https://www.youtube.com/watch?v=bpyLJ9WKLHo)

## 세부 사항
<details>
<summary>접기/펼치기</summary>



</details>
