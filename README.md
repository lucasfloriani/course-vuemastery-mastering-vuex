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

## Mutations and Actions

Mutations podem atualizar/mudar o estado da Vuex
**Mutations são sincronas**

```js
state: {
  count: 0
},
mutations: {
  INCREMENT_COUNT(state) {
    state.count += 1
  }
}
```

```html
<template>
  <button @click="incrementCount">
    Increment
  </button>
</template>

<script>
  export default {
    methods: {
      incrementCount() {
        this.$store.commit("INCREMENT_COUNT");
      }
    }
  };
</script>
```

OBS: **this.\$store.commit** chama a mutation pelo nome

As mutations são escritas todas em maiusculo pois é comum no padrão Flux no qual o Vuex é baseado, porem não é obrigatório, porem ao utilizar este modo, podemos identificar de cara quando são utilizados mutations e as actions em nossos arquivos.

### Dynamic Mutations

Podemos passar parâmetros para nossas mutations com a seguinte sintaxe:

```js
state: {
  count: 0
},
mutations: {
  INCREMENT_COUNT(state, value) {
    state.count += value
  }
}
```

```html
<template>
  <input type="number" v-model.number="incrementBy" />
  <button @click="incrementCount">
    Increment
  </button>
</template>

<script>
  export default {
    data() {
      return {
        incrementBy: 1
      };
    },
    methods: {
      incrementCount() {
        this.$store.commit("INCREMENT_COUNT", this.incrementBy);
      }
    }
  };
</script>
```

### Actions

- **Actions são assincronas**, ou seja, sua ordem é randomica.
- Actions podem embrulhar a lógica de negócio envolta das mutations.
- Actions podem não usar o commit para Mutations em certos casos, pois podem ocorrer certas verificações antes de realizar alguma ação que precise comitar para as Mutations.
- **De acordo com o time core do Vue, sempre deve-se colocar mutations dentro de actions, mesmo que você tenha uma action que é dedicada a uma só mutation, pois assim quando for preciso adicionar código nesta mutation, sua estrutura já estará codificada corretamente (Maior escalabilidade)**

```js
state: {
  user: null,
  count: 0
},
mutations: {
  INCREMENT_COUNT(state, value) {
    state.count += value
  }
},
actions: {
  updateCount({state, commit}, value) {
    if (state.user) {
      commit('INCREMENT_COUNT', value)
    }
  }
}
```

```js
methods: {
  incrementCount() {
    // Chama Action Direto (dispath)
    this.$store.dispatch('updateCount', this.inscrementBy)
    // Chama Mutation Direto (commit)
    // this.$store.commit('INCREMENT_COUNT', this.inscrementBy)
  }
}
```

## Paginação

Para realizar paginação através de parametros get, precisamos adicionar um atributo a tag **router-view** para que ela reconheça que os parametros da url foram trocados e faça o reload da listagem.

```html
<router-view :key="$route.fullPath" />
```

## Vuex Modules

Ao utilizar todos os states, mutations, actions e getters em um arquivo (**store.js**), pode ser complicado manter caso a aplicação cresça.

Utilizando o conceito de **Vuex Modules**, nós podemos separar nossos códigos da store em módulos separados por model (objeto), onde o arquivo principai (store.js) importará o conteudo destes arquivos.

Modules são geralmente criados para:

- **Data models**
- **Features**: funcionalidades que usam state compartilhado, como por exemplo um módulo para autenticação de usuario

Podemos separar a estrutura de pastas como a seguinte:

```text
/store
  /modules
    event.js
    user.js
  store.js
```

Dentro do arquivo do store ao invez de termos as actions, mutations, store e getters, somente teremos a propriedade **modules**:

```js
import * as event from "@/store/modules/event.js";
import * as user from "@/store/modules/user.js";

export default new Vuex.Store({
  modules: {
    event,
    user
  }
});
```

Dentro dos arquivos de modules podemos serguir a sintaxe de constantes, sendo seus beneficios a criação de variaveis e metodos privados:

```js
export const state = {};
export const mutations = {};
export const actions = {};
export const getters = {};
// Para importar usasse
import * as name from "@/store/modules/name.js";
```

Ou tambem podemos utilizar a sintaxe para exportar um objeto:

```js
export default {
  state: {},
  mutations: {},
  actions: {},
  getters: {}
};
// Para importar usasse
import name from "@/store/modules/name.js";
```

OBS: Ambas as sintaxes podem ser utilizadas tranquilamente, somente uma questão de preferencia

É comum acessarmos estados de um module dentro de outro em aplicações.

```js
// event.js
export const actions = {
  createEvent({ commit, rootState }, event) {
    console.log("User creating Event is", rootState.user.user.name);
    return EventService.postEvent(event).then(() => {
      commit("ADD_EVENT", event);
    });
  }
};
```

O parâmetro rootState fornece acesso ao store do arquivo principal do Vuex (store.js)

Para chamarmos actions de um outro modulo executamos da seguinte forma:

```js
// event.js
export const actions = {
  createEvent({ commit, dispatch, rootState }, event) {
    console.log("User creating Event is", rootState.user.user.name);
    dispatch("actionToCall");
    return EventService.postEvent(event).then(() => {
      commit("ADD_EVENT", event);
    });
  }
};
```

Para chamar uma action usamos do mesmo modo que a utilização do arquivo store.js, pois as actions acabam ficando no global namespace.

### Global namespace

**Actions, Mutations and Getters** são sempre registrados dentro do namespace global (Basicamente no root onde o \$store está) mesmo quando usado Modules.

Não importa onde nós os declaramos, eles serão chamados sem o nome do seu módulo.

Vuex foi implementado assim pois nós podemos ter várias actions/mutations usando o mesmo nome, porem com isto pode-se ter problemas com colisão de nomes, algo que o **Vuex Module NameSpacing** resolve

### Vuex Module NameSpacing

```js
// event.js
export const namespaced = true;
```

Garante que todas as mutations, actions e getters estarão dentro do namespace event.
Com isto, precisamos adicionar o prefixo **'nomeDoArquivoModule/NomeDa(Action/Mutation/Getter)**

```js
this.$store.dispatch("event/fetchEvent");
```

Podemos facilitar a utilização de actions em nossos componentes com a **mapActions** do Vuex

```js
export default {
  props: ["id"],
  created() {
    // Pega dispatcher da store que foi adicionada
    // ao methods pela função mapActions
    this.fetchEvent(this.id);
  },
  computed: mapState({
    event: state => state.event.event
  }),
  // methods: mapActions(['event/fetchEvent'])
  //
  // É possível adicionar deste modo,
  // assim todos os itens no array vão ter o prefixo
  // do namespace 'event'
  // Onde (namespace, ActionToMap)
  methods: mapActions("event", ["fetchEvent"])
};
```

Para pegar Getters quando se utiliza namespaces utilizamos da seguinte forma:

```js
// Sem namespacing
computed: {
  getEventById() {
    return this.$store.getters.getEventById
  }
}

// Sem namespacing e com mapGetters
computed: mapGetters(['getEventById'])

// Com namespacing
computed: {
  getEventById() {
    return this.$store.getters['event/getEventById']
  }
}

// Com namespacing e com mapGetters
computed: mapGetters('event', ['getEventById'])
```

Mutations não devem ser chamadas por outros Vuex Modules, onde devem ser somente chamadas pela action dentro do modulo atual, sendo estas boas práticas.

Porem podemos chamar uma action de um outro modulo da seguinte maneira quando utilzado namespace:

```js
// event.js
export const actions = {
  createEvent({ commit, dispatch, rootState }, event) {
    console.log("User creating Event is", rootState.user.user.name);
    // ('nome da action', 'parametros', options)
    dispatch("moduleName/actionToCall", null, { root: true });
    return EventService.postEvent(event).then(() => {
      commit("ADD_EVENT", event);
    });
  }
};
```
