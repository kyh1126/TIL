# Chapter 26. 메인(Main) 컴포넌트

- 메인(Main): 모든 시스템에는 최소한 하나의 컴포넌트가 존재하고, 이 컴포넌트가 나머지 컴포넌트를 생성하고, 조정하며, 관리한다.

## 궁극적인 세부사항

---

- 메인 컴포넌트는 궁극적인 세부사항으로, 가장 낮은 수준의 정책이다.
    - 시스템의 초기 진입점
    - 운영체제를 제외하면 어떤 것도 메인에 의존하지 않는다.
    - 메인은 모든 팩토리와 전략, 그리고 시스템 전반을 담당하는 나머지 기반 설비를 생성한 후, 시스템에서 더 높은 수준을 담당하는 부분으로 제어권을 넘기는 역할을 맡는다.
- 메인에 의존성이 일단 주입되고 나면, 메인은 의존성 주입 프레임워크를 사용하지 않고도 일반적인 방식으로 의존성을 분배할 수 있어야 한다.

- 움퍼스 사냥 게임의 메인 컴포넌트
    - 주목할 부분은 문자열을 로드하는 방법으로, 코드의 나머지 핵심 영역에서 구체적인 문자열을 알지 못하도록 만들었다.
        
        ```java
        public class Main implements HtwMessageReceiver {
          private static HuntTheWumpus game;
          private static int hitPoints = 10;
          private static final List<String> caverns = new ArrayList<>();
          private static final String[] environments = new String[] {
            "bright",
            "humid",
            "dry",
            "creepy",
            "ugly",
            "foggy",
            "hot",
            "cold",
            "drafty",
            "dreadful"
          };
        
          private static final String[] shapes = new String[] {
            "round",
            "square",
            "oval",
            "irregular",
            "long",
            "craggy",
            "rough",
            "tall",
            "narrow"
          };
        
          private static final String[] cavernTypes = new String[] {
            "cavern",
            "room",
            "chamber",
            "catacomb",
            "crevasse",
            "cell",
            "tunnel",
            "passageway",
            "hall",
            "expanse"
          };
        
          private static final String[] adornments = new String[] {
            "smelling of sulfur",
            "with engravings on the walls",
            "with a bumpy floor",
            "",
            "littered with garbage",
            "spattered with guano",
            "with piles of Wumpus droppings",
            "with bones scattered around",
            "with a corpse on the floor",
            "that seems to vibrate",
            "that feels stuffy",
            "that fills you with dread"
          };
        ```
        
    - main 함수: HtwFactory를 사용하여 게임을 생성하는 방식
        - 게임을 생성할 때 htw.game.HuntTheWumpusFacade라는 클래스 이름을 전달하는데, 이 클래스는 메인보다도 더 지저분하기 때문이다.
        - 이 클래스에서 변경이 생겨도 메인을 재컴파일/재배포하지 않아도 되게 하기 위함이다.
        
        ```java
          public static void main(String[] args) throws IOException {
            game = HtwFactory.makeGame("htw.game.HuntTheWumpusFacade", new Main());
            createMap();
            BufferedReader br =
              new BufferedReader(new InputStreamReader(System.in));
            game.makeRestCommand().execute();
            while (true) {
              System.out.println(game.getPlayerCavern());
              System.out.println("Health: " + hitPoints + " arrows: " + game.getQuiver());
              HuntTheWumpus.Command c = game.makeRestCommand();
              System.out.println(">");
              String command = br.readLine();
              if (command.equalsIgnoreCase("e"))
                c = game.makeMoveCommand(EAST);
              else if (command.equalsIgnoreCase("w"))
                c = game.makeMoveCommand(WEST);
              else if (command.equalsIgnoreCase("n"))
                c = game.makeMoveCommand(NORTH);
              else if (command.equalsIgnoreCase("s"))
                c = game.makeMoveCommand(SOUTH);
              else if (command.equalsIgnoreCase("r"))
                c = game.makeRestCommand();
              else if (command.equalsIgnoreCase("sw"))
                c = game.makeShootCommand(WEST);
              else if (command.equalsIgnoreCase("se"))
                c = game.makeShootCommand(EAST);
              else if (command.equalsIgnoreCase("sn"))
                c = game.makeShootCommand(NORTH);
              else if (command.equalsIgnoreCase("ss"))
                c = game.makeShootCommand(SOUTH);
              else if (command.equalsIgnoreCase("q"))
                return;
        
              c.execute();
            }
          }
        ```
        
        - main 함수에서 주목할 점이 더 있다.
            - 입력 스트림 생성 부분, 게임의 메인 루프 처리, 간단한 입력 명령어 해석 등은 main 함수에서 모두 처리하지만, 명령어를 실제로 처리하는 일은 다른 고수준 컴포넌트로 위임한다.
    - 지도 생성 역시 main에서 처리한다.
        
        ```java
          private static void createMap() {
            int nCaverns = (int) (Math.random() * 30.0 + 10.0);
            while (nCaverns-- > 0)
              caverns.add(makeName());
        
            for (String cavern : caverns) {
              maybeConnectCavern(cavern, NORTH);
              maybeConnectCavern(cavern, SOUTH);
              maybeConnectCavern(cavern, EAST);
              maybeConnectCavern(cavern, WEST);
            }
        
            String playerCavern = anyCavern();
            game.setPlayerCavern(playerCavern);
            game.setWumpusCavern(anyOther(playerCavern));
            game.addBatCavern(anyOther(playerCavern));
            game.addBatCavern(anyOther(playerCavern));
            game.addBatCavern(anyOther(playerCavern));
        
            game.addPitCavern(anyOther(playerCavern));
            game.addPitCavern(anyOther(playerCavern));
            game.addPitCavern(anyOther(playerCavern));
        
            game.setQuiver(5);
          }
        
          // 이하 코드 생략
        }
        ```
        

- 요지는 메인은 클린 아키텍처에서 가장 바깥 원에 위치하는, 지저분한 저수준 모듈이라는 점이다.

## 결론

---

- 메인을 애플리케이션의 플러그인이라고 생각하자.
    - 메인은 플러그인이므로 메인 컴포넌트를 애플리케이션의 설정별로 하나씩 두도록 하여 둘 이상의 메인 컴포넌트를 만들 수도 있다.
- 메인을 플러그인 컴포넌트로 여기고, 그래서 아키텍처 경계 바깥에 위치한다고 보면 설정 관련 문제를 훨씬 쉽게 해결할 수 있다.