# Spring

## Spring实战5

- 基础：起步，开发，使用数据，保护，配置

- 集成

  ```
  1. RestTemplate
  2. JmsTemplate,RabbitTemplate
  ```

  

- 反应式Spring

```java
@Test
public void flatMap() {
  Flux<Player> playerFlux = Flux
    .just("Michael Jordan", "Scottie Pippen", "Steve Kerr")
    .flatMap(n -> Mono.just(n)
        .map(p -> {
            String[] split = p.split("\\s");
            return new Player(split[0], split[1]);
          })
        .subscribeOn(Schedulers.parallel())
      );
  List<Player> playerList = Arrays.asList(
      new Player("Michael", "Jordan"),
      new Player("Scottie", "Pippen"),
      new Player("Steve", "Kerr"));
  StepVerifier.create(playerFlux)
      .expectNextMatches(p -> playerList.contains(p))
      .expectNextMatches(p -> playerList.contains(p))
      .expectNextMatches(p -> playerList.contains(p))
      .verifyComplete();
}
```

- 微服务

  ```java
  @Controller
  @RequestMapping("/ingredients")
  public class IngredientController {
    private IngredientClient client;
    @Autowired
    public IngredientController(IngredientClient client) {
      this.client = client;
    }
    @GetMapping("/{id}")
    public String ingredientDetailPage(@PathVariable("id") String id,
                                       Model model) {
      model.addAttribute("ingredient", client.getIngredient(id));
      return "ingredientDetail";
    }
  }
  ```

  

- 部署：管理，监控，部署方案