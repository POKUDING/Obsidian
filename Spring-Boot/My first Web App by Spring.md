## Session vs Request Scopes

### Request Scope
- Active for a single request **ONLY**
	-  리퀘스트에 대한 리스폰스를 보내고 나면 리퀘스트 속성들은 메모리에서 삭제한다.
	- 다음 리퀘스트에서는 이용할 수 없다.
	- 대부분의 경우에 권장된다.
### Session Scop
- 여러 요청에 대한 정보값을 저장할 수 있다.
- 메모리가 낭비될 수 있으니 주의해야한다.

```JAVA
@SessionAttributes("name")
```
위와 같이 SessionAttributes("")어노테이션 안에 저장을 원하는 model의 변수를 넣어주면 해당 값은 SessionScope로 관리되게 된다.
세션 Scope의 값을 공유받고 싶은 Controller에도 같은 어노테이션을 붙여줘야한다.


### Redirect
Todo에서 POST 요청을 처리하고 나서 유저가 todo-list 뷰로 돌아가게 해주고 싶은 경우엔 redirect를 해주면 된다. 그냥 todoList.jsp를 제공할 경우에는 listAllTodos에 있는 로직이 실행되지 않아서 모델이 비워진 상태로 todoList.jsp를 제공하게 된다. 

```JAVA
@RequestMapping("list-todos")  
public String listAllTodos(ModelMap model) {  
    List<Todo> todos = todoService.findByUsername("junhyupa");  
    model.addAttribute("todos", todos);  
    return "listTodos";  
}
```
junhyupa의 이름으로 저장된 todo리스트 todos를 모델에 넣어서  listTodos.jsp에 전달해 주고 있다.

```JAVA
@RequestMapping(value = "add-todo", method = RequestMethod.POST)  
public String addNewTodo(ModelMap model, Todo todo) {  
    String username = (String)model.get("name");  
    todoService.addTodo(username, todo.getDescription(), LocalDate.now().plusYears(1), false);  
    return "redirect:list-todos";  
}
```
만약 return "listTodos" 를 하게될 경우엔 todos가 전달되지 않아서 비어있는 todo-list를 보게될 것이다.
redirect:list-todos 를 하게되면 클라이언트가 /list-todos에 다시 GET요청을 하고 완성된 todoList.jsp뷰를 보게될 것이다.

## Request 를 특정 class에 맞게 전송하는 방법

### 양방향 바인딩
```JAVA
@RequestMapping(value = "add-todo", method = RequestMethod.GET)  
public String showNewTodoPage(ModelMap model) {  
    String username = (String)model.get("name");  
    Todo todo = new Todo(0, username, "", LocalDate.now().plusYears(1), false);  
    model.put("todo",todo);  
    return "todo";  
}
```
새로운 todo를 입력하는 뷰를 요청하면 새로운 Todo 객체를  model에 "todo"로  넣어준다. 이는 todo.jsp의
```jsp
<form:form method="post" modelAttribute="todo">  
    Description: <form:input type="text" path="description" required="required" />  
    <form:input type="hidden" path="id" />  
    <form:input type="hidden" path="done" />  
    <input type ="submit" class="btn btn-success" />  
</form:form>
```
이부분에 매핑되게 된다. 이후 유저가 내용을 수정하여 post요청을 보내게 될 경우 내용이 Todo class로 맵핑 되어 
```JAVA
@RequestMapping(value = "add-todo", method = RequestMethod.POST)  
public String addNewTodo(ModelMap model, Todo todo) {  
    String username = (String)model.get("name");  
    todoService.addTodo(username, todo.getDescription(), LocalDate.now().plusYears(1), false);  
    return "redirect:list-todos";  
}
```
다음과 같이 Todo class로 입력을 받을 수 있다.

### 양방향 바인딩으로의 validation check
```JAVA
  
import jakarta.validation.constraints.Size;

...
@Size(min=10, message="Enter atleast 10 characters")  
private String description;
...
```
Todo class의 description attribute에 @Size 어노테이션으로 해당 필드의 유효 처리를 해줄 수 있다. 이를 실제로 적용시키려면
```JAVA
@RequestMapping(value = "add-todo", method = RequestMethod.POST)  
public String addNewTodo(ModelMap model, @Valid Todo todo, BindingResult result) {  
  
    if(result.hasErrors()) {  
        return "todo";  
    }  
  
    String username = (String)model.get("name");  
    todoService.addTodo(username, todo.getDescription(), LocalDate.now().plusYears(1), false);  
    return "redirect:list-todos";  
}
```
다음과 같이 Todo class 앞에 @Valid 어노테이션을 이용하여 유효처리가 적용될 수 있도록 해주어야 한다.

BindingResult는 현재 유혀검사의 결과를 받게 되는데 만약 error가 발생하였지만 BindingResult를 이용한 처리를 하지 않는다면 유저는 에러페이지를 보게될 것이다. 하지만 다음과 같이 result.hasErrors()로 에러일 경우 todo로 돌아가게 할 경우 유저는 todo 페이지에 머물러 있게 된다.

```jsp<form:form method="post" modelAttribute="todo">  
    Description: <form:input type="text" path="description" required="required" />  
    <form:errors path="description" cssClass="text-warning"/>  
    <form:input type="hidden" path="id" />  
    <form:input type="hidden" path="done" />  
    <input type ="submit" class="btn btn-success" />  
</form:form>
```
또한 form:errors 를 이용하여 @Size 어노테이션에 적어둔 메세지를 화면에 표시해 줄 수 있다.

#의문점 #기술부채 
어떻게 BindingResult안에 결과가 미리 저장되어서 들어갈 수 있었는가?
	- 스프링의 IoC까지 생각을 확장하지 못 했었다. 결국 Spring 컨테이너가 Bean을 관리하고 실행시켜주는 것이기 때문에 요청은 Spring컨테이너가 Bean을 활용하는 방식으로 관리될 것이다. Bean에서 요구하는 파라미터들을 체크하고 어노테이션에 맞게 로직을 실행하고 결과를 BindingResult에 넣어주는 것이다.
form:errors로는 어떻게 데이터가 넘어갈 수 있는가?
jsp의 form: 이란 무엇인가?


