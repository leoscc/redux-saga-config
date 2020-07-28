# Arquitetura Flux e Redux

Hoje aprenderemos os conceitos de arquitetura flux usando o Redux. Mas antes de de colocar o Redux em prática precisamos entender alguns conceitos desta biblioteca, vamos lá:

**O que é arquitetura flux?**

- É uma forma de comunicação entre vários elementos em tela
- Segue um fluxo único

**O que é Redux?**

- Biblioteca que implementa arquitetura flux, pode ser usada com React, Angular, Vue, JavaScript puro e etc

**Para que serve o Redux?**

- Controlar estados globais na aplicação

**Quando usar Redux?**

- Meu estado tem mais de um dono?
- Meu estado é manipulado por mais componentes?
- As ações do usuário causam efeitos colaterais nos dados?

Exemplo: em um carrinho de compras de um e-commerce podemos exibir a quantidade de items no carrinho no header da aplicação, ao adicionar um produto alteramos o estado do carrinho, dentro do carrinho podemos aumentar e diminuir a quantidade de items em locais como este onde o estado do carrinho é "usado" por vários componentes é ideal usar o Redux.

**Princípios**

- Toda _action_ deve possuir um type
- O estado do Redux é o único ponto de verdade
- Não podemos mutar o estado do Redux sem uma action
- As actions e reducers são funções puras, ou seja, não lidam com efeitos colaterais assíncronos (para isto usamos o Redux Saga)
- Qualquer lógica síncrona regra de negócios deve ficar no reducer e não na actions

## Configurando o Redux

> [Este repositório](https://github.com/leon-carvalho/redux-saga-ecommerce-cart/commits/master) contém um projeto pré-configurado que o ajudará a configurar o Redux. Use como base o commit aa27022.

Vamos configurar o Redux em um projeto React, para isto siga os passos:

- Instale o Redux e React Redux: `yarn add redux react-redux`
- Crie uma pasta para guardar todos os arquivos relacionados ao Redux, crie uma pasta chamada: `store` dentro de _src_
- Dentro da pasta _store_ crie um arquivo: `index.js` com a configuração inicial do Redux:

```jsx
// src/store/index.js

import { createStore } from "redux";

const store = createStore();

export default store;
```

- Adicione o `Provider` do Redux dentro de _App.js,_ o provider é responsável por prover o estado do Redux para todos os componentes*:*

```jsx
// App.js
import React from "react";
// importação do provider
import { Provider } from "react-redux";
import { BrowserRouter } from "react-router-dom";

import Header from "./components/Header";

import Routes from "./routes";

import store from "./store";

import GlobalStyles from "./styles/global";

const App = () => {
  return (
    // o Provider precisa receber o store como parâmetro
    // Provider envelopa todos os componentes da aplicação assim todos eles tem acesso ao estado do Redux
    <Provider store={store}>
      <BrowserRouter>
        <Header />
        <Routes />
        <GlobalStyles />
      </BrowserRouter>
    </Provider>
  );
};

export default App;
```

> Se tentarmos rodar a aplicação teremos um erro. Isto acontece pois não podemos criar a _store_ sem nenhum `reducer`. Para resolver este problema vamos criar um reducer para o carrinho

- Crie uma pasta chamada `modules` dentro da pasta _store._
- Dentro de _modules_ crie uma pasta chamada `cart`
- Crie um arquivo chamado `reducer.js` dentro da pasta _cart,_ com o estado inicial do carrinho:

```jsx
// src/store/modules/cart/reducer.js

export default function cart() {
  return [];
}
```

- importe o reducer do carrinho no index da store

```jsx
// src/store/index.js

import { createStore } from "redux";
// reducer do carrinho
import reducer from "./modules/cart/reducer";

// passa o reducer como parametro do createStore
const store = createStore(reducer);

export default store;
```

> ✅ Ao salvar os arquivos e rodar a aplicação tudo deve estar funcionando

Podemos ter várias informações armazenadas no Redux, além do carrinho poderíamos ter a informação de autenticação, de usuário entre outras.

## Adicionando mais de um reducer na aplicação

Vamos configurar a aplicação para lidar com mais de um reducer:

- Crie um arquivo chamado `rootReducer.js` dentro de _src/stores/modules,_ vamos usar o combineReducers para combinar vários reducers em um único reducer:

```jsx
// src/stores/modules/rootReducer.js
import { combineReducers } from "redux";

// reducers
import cart from "./cart/reducer";

// podemos passar uma lista com vários reducers dentro do combineReducers
// se tivessimos um reducer de produto poderiamos fazer:
// export default combineReducers({ cart, products });

export default combineReducers({
  cart,
});
```

- Refatore o código do index da store:

```jsx
// src/store/index.js

import { createStore } from "redux";
// rootReducer
import rootReducer from "./modules/cart/rootReducer";

const store = createStore(rootReducer);

export default store;
```

## Ação de adicionar ao carrinho

Funcionamento desejado: esperamos que ao clicar no botão de adicionar ao carrinho, o objeto contendo titulo, preço e etc seja passado para reducer do carrinho. O reducer do carrinho torna seu estado disponível para todos os componentes da aplicação, assim podemos consumir informações do carrinho no Header por exemplo.

Antes de começarmos precisamos ver alguns conceitos:

Todo reducer recebe por padrão uma variável: `state` e uma variável `action`.

- state: estado anterior antes da alteração
- action: ação disparada contendo o type e o payload

Geralmente os reducers seguem uma estrutura muito parecida entre eles:

```jsx
export default function <nomeDoReducer>(state, action) {
  switch (action.type) {
    case "<ADD_TO_CART>":
      return []; // retorna o estado modificado da maneira como quiser

    default:
      return state; // por padrão retornamos o estado anterior sem alterações
  }
}
```

**Entenda o que fizemos acima:** toda vez que realizamos um `dispatch` todos os reducers são acionados, por exemplo se disparamos uma action de _ADD_TO_CART_ não existe motivos para o reducer relacionado ao usuário ouvir o reducer de cart e disparar uma ação por isto usamos o `switch-case` e retornamos o estado sem alterações por padrão.

### Criando a action de adicionar ao carrinho

- Edite o código da página _Home_ adicionando a função de dispatch do redux para disparar uma ação ao clicar no botão:

```jsx
import React, { useEffect, useState } from "react";
// importa o dispatch do react-redux
import { useDispatch } from "react-redux";
import { MdAddShoppingCart } from "react-icons/md";

import api from "../../services/api";
import { formatPrice } from "../../utils/format";

import { ProductList } from "./styles";

function Home() {
  const dispatch = useDispatch();

  const [products, setProducts] = useState([]);

  // envia para o redux o id do produto
  const handleAddProduct = (product) => {
    dispatch({
      type: "ADD_TO_CART",
      payload: {
        product,
      },
    });
  };

  useEffect(() => {
    async function loadProducts() {
      const response = await api.get("products");

      const data = response.data.map((product) => ({
        ...product,
        formattedPrice: formatPrice(product.price),
      }));

      setProducts(data);
    }

    loadProducts();
  }, []);

  return (
    <ProductList>
      {products.map((product) => (
        <li key={product.id}>
          <img src={product.image} alt={product.title} />
          <strong>{product.title}</strong>
          <span>{product.formattedPrice}</span>
          {/* dispara a action no evento de clique do botão */}
          <button type="button" onClick={() => handleAddProduct(product)}>
            <div>
              <MdAddShoppingCart size={16} color="#fff" />
            </div>
            <span>ADD TO CART</span>
          </button>
        </li>
      ))}
    </ProductList>
  );
}

export default Home;
```

- edite o reducer do carrinho para que ele adicione o novo item ao estado:

```jsx
// estado inicial
const INITIAL_STATE = [];

// dispara a ação caso o type seja: ADD_TO_CART, adicionando um novo item ao carrinho
export default function cart(state = INITIAL_STATE, action) {
  switch (action.type) {
    case "ADD_TO_CART":
      return [...state, action.payload.product];
    default:
      return state;
  }
}
```

## Consumindo dados do Redux

Agora que sabemos como adicionar informações no estado do Redux vamos aprender a consumir estas informações e é muito fácil!

- acesse o componente que consumirá a informação do Redux
- import o `useSelector()` de dentro do _react-redux,_ com ele conseguimos extrair dados de dentro da store do redux

Confira um pequeno exemplo, abaixo:

```jsx
import React from "react";
// importa o useSelector
import { useSelector } from "react-redux";

export const CounterComponent = () => {
  // acessa a informação do estado do reducer counter
  const counter = useSelector((state) => state.counter);

  return <div>{counter}</div>;
};
```

## Actions

Uma prática comum no Redux é separar as actions disparadas nos componentes em um novo arquivo já que cada action está associada com um modulo da aplicação.

Por exemplo podemos agrupar todas as actions relacionadas ao carrinho em um único arquivo.

- Crie um arquivo chamado `action.js` dentro da pasta _src/store/modules/cart/actions.js_
- As actions do carrinho ficariam parecidas com isto:

```jsx
//src/store/modules/cart/actions
export function addToCart(product) {
  return {
    type: "@cart/ADD",
    payload: { product },
  };
}

export function removeFromCart(id) {
  return {
    type: "@cart/REMOVE",
    payload: { id },
  };
}

// É uma boa prática nomear o type das actions com o módulo: @cart/ ação: REMOVE
```

- agora importamos as actions deste arquivo quando precisarmos disparar alguma delas:

```jsx
// src/pages/Cart.js

// importa as actions, usamos o * pois temos mais de um export
import * as CartActions from "../../store/modules/cart/actions";

// dispara uma action
dispatch(CartActions.removeFromCart(product.id));
```

Além de tornar o código mais organizado, agora que separamos as actions conseguimos utilizar a action de adicionar ao carrinho em vários componentes de uma forma mais fácil.

## Debugando Redux com Reactotron

...todo

## Configurando Redux Saga

A partir de agora, vamos trabalhar com o conceito de `middlewares` dentro do Redux.

**O que é um middleware?** É um interceptador, no Redux o middleware é capaz de interceptar actions. Quando disparamos uma action um middleware pode entrar em ação realizando algum efeito colateral como uma chamada assíncrona na API, salvar um item no storage da aplicação entre outros.

**Instalando Redux Saga**

- Adicione a dependência do redux-saga: `yarn add redux-saga`
- Crie um arquivo chamado `sagas.js` dentro da pasta module do arquivo que será usado. Ex: store/modules/cart/actions.js
- Edite o arquivo de actions, perceba que usamos uma sintaxe diferente neste arquivo. Quando lidamos com Redux Saga usamos a sintaxe de **generators,** que podemos pensar como uma extensão do _async/await_

Vamos usar uma função cuja responsabilidade é acessar a api buscar os detalhes do produto e retornar para o carrinho

```jsx
//src/store/modules/cart/actions.js
function* addToCart() {}

// * = async
// function* = async function
// yield = await
```

No redux saga não podemos simplesmente usar `api.get()` devemos usar alguns métodos especiais de dentro de redux-saga/effects

- `call()` - responsável por chamar métodos assíncronos que retornam promises. Usamos para fazer uma chamada na API por exemplo.
- `put()` - dispara uma action de dentro do saga
- `select()` - obtêm um valor de dentro do estado do Redux

```jsx
// exemplo com call()
const response = yield call(api.get, `/products/${id}`);
```

exemplo com put():

```jsx
import { addToCartSuccess } from './actions';

yield put(addToCartSuccess(response.data));
```

Como adicionamos um passo a mais no fluxo dos dados com o Redux Saga, vamos dividir as nossas actions em dois tipos: `action.SUCCESS` e `action.REQUEST` assim precisamos refatorar os arquivos que disparam a action para disparar a action.REQUEST, o arquivo de action criando uma action de request e uma de success.

```jsx
// actions.js
export function addToCartRequest(id) {
  return {
    type: "@cart/ADD_REQUEST",
    id,
  };
}

export function addToCartSuccess(product) {
  return {
    type: "@cart/ADD_SUCCESS",
    product,
  };
}
```

Também precisamos criar um arquivo chamado `rootSaga` para agrupar os sagas da nossa aplicação.

## Persistindo dados com Redux Persist

- Adicione a lib: `yarn add redux-persist`
- Crie um arquivo chamado `persistReducers.js` dentro da pasta _store_

```jsx
// src/store/persistReducers

// storage: pega a estratégia de storage padrão do ambiente ex: localStorage, AsyncStorage...
import storage from "redux-persist/lib/storage";
import { persistReducer } from "redux-persist";

export default (reducers) => {
  const persistedReducer = persistReducer(
    {
      key: "gobarber", // identifica a nossa aplicação
      storage,
      whitelist: ["auth", "user"],
      // passamos na whiteList os nomes dos reducers que queremos armazenar informações
    },
    reducers
  );

  return persistedReducer;
};
```

- Edite o index da store, passando as informações do Redux Persist

```jsx
// src/store/index.js
import { persistStore } from "redux-persist";
//...
import persistReducers from "./persistReducers"; // importa o arquivo que criamos acima

// passamos o persistReducers dentro do store
const store = createStore(persistReducers(rootReducer), middlewares);
const persistor = persistStore(store);

export { store, persistor }; // altera o import,já que agora temos mais de um
```

- Edite os arquivos que importam a store, como por exemplo o App.js:

```jsx
// src/App.js

import React from "react";
import { PersistGate } from "redux-persist/integration/react"; // simalar ao provider
//...
import { store, persistor } from "./store";

// como os componentes estão dentro do PersistGate as rotas só serão renderizadas
// após buscar as informações de dentro do storage

export default function App() {
  return (
    <Provider store={store}>
      <PersistGate persistor={persistor}>
        <Router>{/*...*/}</Router>
      </PersistGate>
    </Provider>
  );
}
```

Agora o storage já deve estar funcionando quando recarregamos a página.
