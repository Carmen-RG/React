# 2- Introduccion a la renderización de elementos en el DOM (React)

Los nodos HTML cuyo contenido vaya a ser manejado por el DOM de React se llaman nodos raíz del DOM (*root DOM node*). Las aplicaciones desarrolladas con React suelen tener un único nodo raíz del DOM, pero puede haber tantos nodos raíz como se quiera.

## 1. Renderización de un elemento

Para renderizar un elemento de React:

0. Suponiendo que en el HTML (y por tanto en el DOM) haya un `<div>` de id "root":  

        <div id="root"></div>

1. Pasar el elemento del DOM como parámetro de `ReactDOM.createRoot()` para crear el nodo raíz del DOM. En el ejemplo siguiente lo llamamos `root`, pero puede tener cualquier otro nombre.

        const root = ReactDOM.createRoot(
        document.getElementById('root')
        );

2. Pasar el elemento de React como parámetro de `<elemento-raiz>.render()`. En este caso, al haber llamado al nodo raíz `root`, es `root.render()`:

        const element = <h1>Hello, world</h1>;

        root.render(element);

En conjunto:

    const root = ReactDOM.createRoot(
        document.getElementById('root')
    );
    const element = <h1>Hello, world</h1>;
    root.render(element);


## 2. Actualización del elemento renderizado
Los elementos de React son inmutables: representan la UI en un determinado momento. Una vez creado, no puede modificarse.

La actualización de la UI puede hacerse creando un nuevo elemento y renderizándolo, pero lo óptimo es hacerlo mediante componentes.

### 2.1 Componentes
Conceptualmente un componente es como una función de JavaScript. Aceptan parámetros (llamados *props*, de propiedades) y devuelven elementos de React que describen la UI.

#### 2.1.1 Definición de un componente
Hay dos formas de definir un componente:

- Con una función de JavaScript que devuelva el elemento. Por ejemplo:

        function Welcome(props) {
            return <h1>Hello, {props.name}</h1>;
        }

- Con una clase de JavaScript que herede de la clase `React.Component` e implemente la función `render()`, que devuelve el elemento. Por ejemplo:

        class Welcome extends React.Component {
            render() {
                return <h1>Hello, {this.props.name}</h1>;
            }
        }


#### 2.1.2 Renderizar un componente
**A tener en cuenta**: los elementos de React pueden representar, además de etiquetas del DOM, componentes definidos por los usuarios (*user-defined components*, en adelante componentes UD). Estos componentes siempre deben empezar con letra mayúscula, a diferencia de los componentes ya definidos del DOM, que empiezan por minúscula. Así, `<div />` representa una etiqueta HTML, pero `<Welcome />` representa un componente y requiere que `Welcome` esté definido en ese ámbito.

Un ejemplo de componente UD:

    const element = <Welcome name="Sara" />;

React actúa de la siguiente forma respecto a los elementos que representan componentes UD: pasa los atributos JSX y los hijos a este componente como un único objeto. Este objeto se llama *props*.

**A tener en cuenta:** un componente, sin importar la forma en que se declare, no debe modificar nunca sus propias *props*.

Por ejemplo, suponiendo que en el HTML exista un elemento `<div id="root"></div>`, el siguiente código renderiza "Hello, Sara":

    function Welcome(props) {
        return <h1>Hello, {props.name}</h1>;
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    
    const element = <Welcome name="Sara" />;
    
    root.render(element);

El proceso es:

1. Llamamos a `root.render()` con el elemento `<Welcome name="Sara" />`.

2. React llama al componente `Welcome` con los *props* `{name: 'Sara'}`.

3. Nuestro componente `Welcome` devuelve el elemento `<h1>Hello, Sara</h1>`.

4. El DOM de React actualiza el DOM para mostrar `<h1>Hello, Sara</h1>`.


#### 2.1.3 Componer componentes
Los componentes pueden hacer referencia a otros componentes en su output. 

Un ejemplo de un componente `App` que renderiza `Welcome` varias veces:

    function Welcome(props) {
        return <h1>Hello, {props.name}</h1>;
    }

    function App() {
        return (
            <div>
                <Welcome name="Sara" />
                <Welcome name="Cahal" />
                <Welcome name="Edite" />
            </div>
        );
    }

Esto también implica que podemos modularizar los componentes, dividiéndolos en componentes más pequeños.

#### 2.1.4 Estados de un componente declarado en una clase
El estado (`state`) es similar a las *props*, pero es privado y el componente lo controla completamente. El estado es un objeto que, al cambiar, provoca que el componente vuelva a renderizarse. Esta es la forma óptima de actualizar un componente.

Conceptualmente, el estado de un componente es el equivalente a una variable local de una función, y las *props* son el equivalente a los parámetros de una función.

Para utilizar los estados correctamente es necesario tener en cuenta:

- El estado debe modificarse mediante la función `setState()`.

- Las actualizaciones de `this.props` y `this.state` pueden ser asíncronas, por lo que no deberíamos apoyarnos en sus valores para calcular el siguiente estado. Podemos utilizar una versión de `setState()` que acepta una función como parámetro en vez de un objeto; esa función recibirá como parámetros el estado anterior y las *props* en el momento de la actualización. Por ejemplo:

        this.setState(function(state, props) {
            return {
                counter: state.counter + props.increment
            };
        });

- Las actualizaciones de estados se fusionan. `setState()` fusiona el objeto que le proporcionamos como parámetro con el estado actual. Si el estado contiene varias variables, podemos actualizarlas de forma independiente.

- Los estados solo son accesibles para el componente al que pertenecen. Este puede elegir pasarlo como *prop* a sus componentes hijos, pero a nada más. Es un flujo de datos unidireccional, de arriba a bajo.


#### 2.1.5 Ciclo de vida de un componente declarado en una clase

En cuanto al ciclo de vida, si nuestra aplicación tiene muchos componentes, es importante liberar los recursos utilizados por estos cuando se destruyan. 

- El *mounting* ("montaje") se produce la primera vez que un componente se renderiza en el DOM. La función `componentDidMount()` se ejecuta después del *mounting*. 

- El *unmounting* ("desmontaje") se produce cuando el DOM producido por el componente se elimina. La función `componentWillUnmount()` se ejecuta después del *unmounting*.

La forma correcta de actualizar el renderizado de un elemento es utilizando su `state` y liberando los recursos con `componentDidMount()` y `componentWillUnmount()`.