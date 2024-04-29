Controller단에서 LocalDate, LocalDateTime을 사용하는 경우, 직렬화 과정 쉽게 처리하도록 사용.

> **@JsonFormat
-** JSON 출력에 대한 필드 또는 속성의 형식을 지정하는데 사용되는 Jackson 어노테이션.
> 

> **@DateTimeFormat
> -** 스프링에서 지원하는 어노테이션
> 

### 사용 경우

### **@ModelAttribute, @ReqeustParam**

- DateTimeFormat 사용 가능
- @JosnFormat 사용 불가능

### **@RequestBody**

- @DateTimeFormat 사용 가능
- @JosnFormat 사용 가능

<aside>
💡 그렇다면 둘다 존재할때의 우선순위는?

</aside>

두 어노테이션이 존재하면 @JsonFormat이 진행된다. 왜냐하면 스프링의 기본 JSON 컨버터는 Jackson이다. 

그러다보니 JSON으로 변활을 할 때 Jackson을 통해서 진행된다. 

만약 직렬화 과정에서 @JosnFormat이 없다면 스프링에서는 @DateTimeFormat을 통해 직렬화를 진행한다. 

Jackson은 스프링의 어노테이션인 @DateTimeFormat을 전혀 알 수 없다. 그렇기에 @JsonForma이 틀리면 @DateTimeFormat이 맞더라도 직렬화는 실패한다. 

Json 직렬화 외에는 Jackson이 사용되지 않기 때문에 @JsonFormat은 효과가 없다. 

- `@RequestParam`나 `@ModelAttribute`는 HTTP 요청의 쿼리 매개변수나 폼 데이터를 처리하는 데 사용되므로, JSON 직렬화나 역직렬화와 직접적인 관련이 없다. 따라서 `@JsonFormat`은 이러한 애노테이션과 함께 사용되지 않는다.

### **Response Body**

응답으로 JSON 값을 리턴하는 경우에는 @JsonFormat만 사용이 가능하다. 왜냐하면 위와 같은 이유 때문이다.

