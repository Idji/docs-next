# Регистрация компонентов

> Подразумевается, что уже изучили и разобрались с разделом [Основы компонентов](component-basics.md). Если нет — прочитайте его сначала.

## Именование компонентов

При регистрации компонента ему всегда будет присваиваться имя. Например, при глобальной регистрации, с которой встречались ранее:

```js
const app = Vue.createApp({...})

app.component('my-component-name', {
  /* ... */
})
```

Имя компонента указывается первым аргументом `app.component`. В примере выше именем компонента станет "my-component-name".

Имя, которое даёте компоненту, может зависеть от того, где собираетесь его использовать. При использовании компонента непосредственно в DOM (в отличие от строковых шаблонов или [однофайловых компонентов](../guide/single-file-component.md)) настоятельно рекомендуем следовать [правилам W3C](https://html.spec.whatwg.org/multipage/custom-elements.html#valid-custom-element-name) по именованию пользовательских тегов:

1. Все символы в нижнем регистре
2. Содержат дефис (т.е. состоит из нескольких слов соединённых символом дефиса)

Это позволит избежать конфликтов с существующими и будущими HTML-элементами.

Другие советы по именованию компонентов можно узнать в разделе [рекомендаций](../style-guide/#именование-базовых-компонентов-настоятельно-рекомендуется).

### Стиль именования

При описании компонентов в строковых шаблонах или однофайловых компонентах есть два способа их именования:

#### В стиле kebab-case

```js
app.component('my-component-name', {
  /* ... */
})
```

При объявлении компонента в стиле kebab-case, также нужно использовать kebab-case и при обращении к пользовательскому элементу, например так `<my-component-name>`.

#### В стиле PascalCase

```js
app.component('MyComponentName', {
  /* ... */
})
```

При объявлении компонента в стиле PascalCase, можно использовать любой стиль именования при обращении к пользовательскому элементу. А значит `<my-component-name>` и `<MyComponentName>` являются допустимыми вариантами. Но запомните, что только имена в kebab-case являются допустимыми при использовании непосредственно в DOM (т.е. в не строковых шаблонах).

## Глобальная регистрация

До сих пор компоненты создавались с помощью `app.component`:

```js
Vue.createApp({...}).component('my-component-name', {
  // ... опции ...
})
```

Такие компоненты **регистрируются глобально** для приложения. Это означает, что они могут использоваться в шаблоне любого экземпляра компонента в этом приложении:

```js
const app = Vue.createApp({})

app.component('component-a', {
  /* ... */
})
app.component('component-b', {
  /* ... */
})
app.component('component-c', {
  /* ... */
})

app.mount('#app')
```

```html
<div id="app">
  <component-a></component-a>
  <component-b></component-b>
  <component-c></component-c>
</div>
```

Это относится и ко всем дочерним компонентам — все три этих компонента будут также доступны _внутри каждого из них_.

## Локальная регистрация

Глобальная регистрация обычно не лучший способ. Например, при использовании системы сборки, такой как Webpack, глобальная регистрация всех компонентов означает, что даже если перестали использовать компонент, он всё равно будет включён в сборку приложения. Это лишь увеличит количество JavaScript, который потребуется загружать пользователям.

В таких случаях можно определять компоненты как обычные объекты JavaScript:

```js
const ComponentA = {
  /* ... */
}
const ComponentB = {
  /* ... */
}
const ComponentC = {
  /* ... */
}
```

А затем в опции `components` указывать компоненты, которые будут использоваться:

```js
const app = Vue.createApp({
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```

В каждом свойстве объекта `components`, ключ будет определять имя пользовательского элемента, а значение — объект, содержащий опции компонента.

Обратите внимание, что **локально зарегистрированные компоненты _будут недоступны_ в дочерних компонентах**. Например, если `ComponentA` должен быть доступен в `ComponentB`, потребуется его в нём и зарегистрировать:

```js
const ComponentA = {
  /* ... */
}

const ComponentB = {
  components: {
    'component-a': ComponentA
  }
  // ...
}
```

При использовании модулей ES2015 через Babel и Webpack это может выглядеть так:

```js
import ComponentA from './ComponentA.vue'

export default {
  components: {
    ComponentA
  }
  // ...
}
```

Обратите внимание, что в ES2015+ указание имени переменной, такой как `ComponentA` внутри объекта, является сокращением для `ComponentA: ComponentA`, что означает имя переменной будет:

- именем пользовательского элемента для использования в шаблоне, и
- именем переменной, содержащей опции компонента

## Модульные системы

Если не используете модульных систем с `import`/`require` — пока можно пропустить этот раздел. Если применяете, то в таком случае есть несколько советов.

### Локальная регистрация в модульной системе

Чаще всего используют модульную систему, такую как Babel и Webpack. В этих случаях рекомендуем создать каталог `components`, где каждый компонент будет в своём файле.

Затем нужно будет импортировать каждый компонент, который хотите использовать, прежде чем его зарегистрировать локально. Например, в гипотетическом файле `ComponentB.js` или `ComponentB.vue`:

```js
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  }
  // ...
}
```

Теперь в шаблоне `ComponentB` можно использовать оба: `ComponentA` и `ComponentC`.
