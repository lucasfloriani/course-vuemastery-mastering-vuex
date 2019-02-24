# Mastering Vuex

É uma ferramenta para gerenciar estado (dados) dentro de uma aplicação Vue, foi baseado na padrão Flux criado pelo Facebook.

Foca em colocar toda a informação em um só local, facilitando assim a transmissão de informações entre os componentes (**Single Source of Truth**).

Todos os componentes tem acesso direto a ele (Vuex).

O State é reativo, o que significa que quando alguem altera o estado, os componentes que o utilizam tambem recem esta atualização.

Vuex fornece um padrão para alteração e leitura deste estado, simplificando o gerenciamento de dados "globais".

```js
// Criando uma instância Vue
const app = new Vue({
  data: {}, // dados
  methods: {}, // usados para atualizar dados (data)
  computed: {} // Usado para transformar dados em novas variáveis
});
// Criando uma instância Vuex
const store = new Vuex.Store({
  state: {}, // igual data do Vue
  actions: {}, // igual methods do Vue, porem atualizam o state
  getters: {}, // Igual computed do Vue
  mutations: {} // Entregar e rastrear mudanças de estado
});
```

OBS: É uma prática comum as **actions** chamarem as **mutations** onde alteram o **state** diretamente.
OBS2: Com este padrão, com a extensão do Vue do navegador, podemos voltar as alterações da **mutation** para realizar o debug mais facilmente.

Exemplo de Vuex completo:

```js
const store = new Vuex.Store({
  state: {
    loadingStatus: "notLoading",
    todos: [
      { id: 1, text: '...', done: false },
      { id: 2, text: '...', done: true },
      { id: 3, text: '...', done: true },
    ]
  },
  mutations: {
    SET_LOADING_STATUS(state, status) {
      state.loadingStatus = status;
    },
    SET_TODOS(state, todos) {
      state.todos = todos;
    }
  },
  actions: {
    fetchTodos(context) {
      context.commit("SET_LOADING_STATUS", "loading");
      axios.get("/api/todos").then(response => {
        context.commit("SET_LOADING_STATUS", "notLoading");
        context.commit("SET_TODOS", response.data.todos);
      });
    }
  },
  getters: {
    doneTodos(state) {
      return state.todos.filter(todo => todo.done),
    }
  }
});
```

Para utilizarmos as actions(alterar o state) nos componentes usamos o seguinte como comando:

```js
this.$store.dispatch("nomeDaAction");
// this.$store.dispatch('fetchTodos')
```

Para pegar os dados (getter) utilizamos o seguinte comando:

```js
this.$store.getters.nomeDoGetter;
// this.$store.getters.doneTodos
```

Dentro da extensão do Vue no navegador, podemos ver o tempo gasto de cada uma das mutations.

## State and Getters

Para pegarmos os valores do state podemos utilizar os seguintes tipos:

```js
// mapeando diretamente os valores
export default {
  computed: {
    userName() {
      return this.$store.state.user.name;
    },
    userID() {
      return this.$store.state.user.id;
    }
  }
};
```

```js
// Usando mapState para facilitar a pegar o state
import { mapState } from "vuex";

export default {
  computed: mapState({
    userName: state => state.userName,
    userID: state => state.user.id,
    categories: state => state.categories
  })
};
```

```js
// Versão simplificada do mapState
// Quando valores são filhos diretos do state,
// é possivel adicionar somente o nome da propriedade
import { mapState } from "vuex";

export default {
  computed: mapState({
    user: "user",
    categories: "categories"
  })
};
```

```js
import { mapState } from "vuex";

export default {
  computed: mapState(["user", "categories"])
};
```

Caso seja preciso adicionar outras computed properties junto com as do state, podemos utilizar a funcionalidade de Object Spread do javascript como abaixo:

```js
import { mapState } from "vuex";

export default {
  computed: {
    localComputed() {
      return "something";
    },
    ...mapState(["user", "categories"])
  }
};
```

As vezes em nossa aplicação podemos precisar que os valores de um getters seja utilizado em outro getters em nossa store.

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: "...", done: false },
      { id: 2, text: "...", done: true },
      { id: 3, text: "...", done: true }
    ]
  },
  mutations: {},
  actions: {},
  getters: {
    doneTodos(state) {
      return state.todos.filter(todo => todo.done);
    },
    // Aqui é utilizado a função doneTodos do próprio getter
    activeTodosCount: (state, getters) => {
      return state.todos.length - getters.doneTodos.length;
    }
  }
});
```

Podemos tambem utilizar o getter de outra maneira:

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: "...", done: false },
      { id: 2, text: "...", done: true },
      { id: 3, text: "...", done: true }
    ]
  },
  mutations: {},
  actions: {},
  getters: {
    doneTodos(state) {
      return state.todos.filter(todo => todo.done);
    },
    // Assim não é preciso utilizar outro getter para isto
    // simplificando assim a função.
    activeTodosCount: state => {
      return state.todos.filter(todo => !todo.done).length;
    }
  }
});
```

Para passar parâmetros para um getter utilizamos a seguinte sintaxe:

```js
const store = new Vuex.Store({
  state: {
    events: [
      { id: 1, title: "...", organizer: "..." },
      { id: 2, title: "...", organizer: "..." },
      { id: 3, title: "...", organizer: "..." }
      { id: 3, title: "...", organizer: "..." }
    ]
  },
  mutations: {},
  actions: {},
  getters: {
    getEventById: state => id => {
      return state.events.find(events => event.id === id)
    }
  }
});
```

Com isto podemos utilizar em nosso componente da seguinte maneira:

```js
export default {
  computed: {
    getEvent() {
      return this.$store.getters.getEventById;
    }
  }
};
```

```html
<p>{{ getEvent(1) }}</p>
<!--{ id: 1, title: "...", organizer: "..." }-->
```

Para pegar os valores getters da store mais facilmente podemos utilizar a função **mapGetters** do **vuex**

```js
import { mapState, mapGetters } from "vuex";

export default {
  computed: {
    ...mapGetters(["getEventById"]),
    ...mapState(["user", "categories"])
  }
};
```
