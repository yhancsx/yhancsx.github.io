- ... having business rules isolated from framework-specific things. This means that our core logic is not coupled to React, React Native, Express, etc...  
비지니스룰은 react, react-native, express와 같은 외부 framework와 분리되어야 한다.
- This gives you enough flexibility to, for example, move specific parts of the application to a backend, change libraries without too much pain, test once and reuse as many times as you want, share code between React and React Native applications, among others.  
분리된 레이어는 특정 부분을 백엔드로 넘기거나 라이브러리를 교체할 때 많은 수정 사항을 요하지 않고 유연하게 대처 가능하다.
- What I mean by that is that we should organize our codebase around the business rules and not around the frameworks we use to achieve business rules.  
코드 베이스를 프레임워크 기준이 아닌 비지니스 로직 기준으로 작성해야한다.
- The inner circles must not know about the outer circles. That is, there cannot be an import of a use case within an entity, or import of a framework within a use case  
내부의 원은 외부의 원에서 어떤일이 일어나는지 모른다. 외부 원의 코드를 import 하는 일이 없어야 한다.
- Entities and use cases should not rely on external libraries  
엔티티와 그를 감싼 어플리케이션 유스케이스는 외부 라이브러리에 의존하지 말아야 한다.
- The core of our application must be robust enough and malleable enough to meet the demands of the business without needing any external intervention
우리 애플리케이션의 핵심 부분은 외부 개입 없이 비즈니스의 요구를 충족할 수 있을 만큼 충분히 강력하고 유연해야 한다.

- **Entity:** Application independent business rules
- **Interactor(useCase):** Application-specific business rules
- **Adapter:** Glue code from/to Interactors and Presenter, most of the time implementing a framework-specific behaviour. e. g.: We have to connect Interactor with react container, to do so, we have to connect Interactor with redux (framework) and then connect redux to container components.
- **Presenter:** Maps data from/to Adapter to/from Components.
- **Components:** Simplest possible unit of presentation. Any mapping, conversion, MUST be done by the Presenter.

### 1. Counter App
- Entity: Counter class (count, increament, decrement)
- Use Case: wrapping counter (cou,t increment, decrement, top/bottom boundaries)
- Adapter: reducer (action, state)
- Framework: redux store setup
- Presenter: wrapping component (connect dispatch functions) 
- Component: Counter component (get count, actions as props)

### 2. Authentication
- Entity: User class (email, name), Credential class (email, password)
- Use Case: Sign up / Sign In
- Adapter: reducer
- Service: http request
- Framework: redux store setup
- Presenter: wrapping component (connect dispatch functions) 
- Component: Counter component (user, signin, signup, signout component)