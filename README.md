#  API Hacker News

En el primer video aprendimos a hacer una lista dentro de un navigation view, en la cual dicha lista se alimenta dinamicamente a partir de una estructura creada por aparte la cual contiene un id y un título. Dicha lista la hicimos de tipo Identifiable. Quedando el código de la siguiente manera:

import SwiftUI

struct ContentView: View {
    var body: some View {
        NavigationView {
            List(posts) { post in
                Text(post.title)
            }
            .navigationTitle("H4X0R NEWS")
        }
    }
}
struct Post: Identifiable {
    let id: String
    let title: String
}

let posts = [
    Post(id: "1", title: "Hello"),
    Post(id: "2", title: "Bonjour"),
    Post(id: "3", title: "Hola")
]

Entonces con el código anterior salen 3 filas, y asi se llena de manera dinamica las listas, a partir de un arreglo.

##Segundo Video:

En esta leccion vamos a crear la clase para hacer la peticion al url traer la info y guardarla en una variable.

1. Primero creamos un nuevo archivo swift y ahi declaramos una clase

2. Dentro creamos una funcion "fechData" que será con la que obtengamos los datos del servidor.

3. Dentro de la funcion creamos una constante de tipo URL con el string del link de la API. A esta misma constante le aagregamos un if ya que es una constante opcional (no entendi muy bien eso pero se supone que si es opcional le tenemos que agregar un if por si no trae informacion el link)

4. Dentro del if vamos a declarar otra constante de tipo URLSession

5. Despues declaramos otra constante que va a ser igual a la constante anterior, esto para asignarle una tarea (datatask) y que reciba los parametros data, response y error. Dentro de esta variable se crea otro if por que es de tipo opcional, entonces como lo comente antes debe llevar un if

6. el if llevara si el error es igual a nulo entonces: (si es igual a nulo quiere decir que no contiene información)

7. Declaramos otra constante "decoder" y lo igualamos a JSONDecoder()

8. Despues otro if: si tiene informacion data entonces Decodificamos:

9. Para decodificar tenemos que envolverlo en un DO/CATCH, y ademas usar la palabra try, y ademas igualarlo a una constante la cual sera la que guarde todos los datos ya descodificados. En el catch solo imprimimos el error

10. Por ultimo a la variable que contiene la dataTask le damos resume() para que se active.

class NetworkManager {
    func fetchData() {
        if let url = URL(string: "https://hn.algolia.com/api/v1/search?tags=front_page") {
            let session = URLSession(configuration: .default)
            let task = session.dataTask(with: url) { (data, response, error) in
                if error == nil {
                    let decoder = JSONDecoder()
                    if let safeData = data {
                        do {
                            let results = try decoder.decode(Results.self, from: safeData)
                        }catch {
                            print(error)
                        }
                    }
                }
            }
            task.resume()
        }
    }
}

** Olvide agregar el paso donde agregamos un nuevo archivo swift donde pondremos la estructura de los datos JSON, esto se hace antes de descodificar el data:
1. crear archivo
2. declarar una estructura "Results" de tipo Decodable, contendrá una constante "hits" de tipo estructura por que asi viene el JSON
3. Creamos la estructura que mencionamos en el punto anterior con las constantes del json: objectID,points, title, url.
    Debe de ser tipo decoable e Identifiable, y como es Identifiable debemos darle un id pero ese id lo igualamos al parametro objectID:
    
struct Results: Decodable {
    let hits: [Post]
}

struct Post: Decodable, Identifiable {
    var id: String {
        return objectID
    }
    let objectID: String
    let points: Int
    let title: String
    let url: String
}

*Con todo el codigo anterior ya tenemos en la constante results todos los datos descodificados del JSON para poder usarlos.

##Ahora ¿como vamos a llevar esos datos guardados que actualmente estan en la clase NetworkManager a nuestro ContentView

En esta nueva leccion escribiremos la logica para mostrar la informacion en pantalla:

1. primero hacemos nuestra clase "NetworkManager" que conforme el tipo ObservableObject, con esto puede comenzar a transmitir una o muchas de sus propiedades a cualquier vista que lo requiera.

2. Despues declaramos una variable "posts" y la inicializamos que sea de tipo [Post]() la cual es la estructura que hicimos en nuestro NewsModel

3. Despues igualamos la variable anterior a la informacion que guardamos en nuestra constante "results" que esta dentro del DO. A esta variabe le ponemos el sufijo self.posts = results.hits

4. Luego convertimos nuestra variable posts en un propertywrapper @Published

5. ya con eso podemos usar esa variable en cualquier vista.
    5.1 Para ello nos vamos a la vista donde lo queremos (en este caso es el ContentView)
    5.2 creamos un objeto del NetworkManager, lo llamamos igual solo con la primera en minuscula
    5.2 al objeto anterior le agregamos un propertywrapper llamado @ObservedObject, de esta manera nos suscribimos a las actualizaciones del NEtworkManager, entonces cada vez que se actaliza, se ejecuta este propertywrapper y con eso se actualizan los datos
    5.3 Con lo anterior ya podemos utilizar la variable en nuestra list(networkManaer.posts)
    
Ahora solo nos falta una cosa. Nota que ¿a donde llamamos realmente esta funcion fetchData?¿Donde podemos hacer eso en nuestro contentview?
Para realizar esto podemos mandar a llamar a nuestro metodo onAppear() entonces aqui ejecutamos nuestra funcion fetchData
1. creamos el metodo onAppear despues del NavigatinView y dentro del metodo activamos la funcion; y debido a que estamos dentro de un closure, tenemos que agregarle el sufijo self

Ahora solo nos falta una última cosa por hacer, que es el punto cuando usamos @Published, siempre tenemos que asegurarnos de obtener el hilo principal. Paa ello tenemos que utilizar el DispatchQueu y dentro de ese closure vamos a establecer la propiedad @Published. Si no lo hcemos vamos a obtener un error en la consola y esto es porque esta propiedad es una propiedad que otros objetos están escuchando. Esta actualización debe realizarse en el hilo principal. Si sucede en segundo plano, nuestro ContentView será notificada de manera oportuna.

¡ Y listo !

Si corremos nuestra aplicacion vemos que no se está renderizando la informacion, y el error que nos manda la consola es valueNotFound, entonces si vemos en nuestro JSON vemos que la propiedad url en realidad dice null entonces nuestro codigo ejecuta el codigo CATCH
Para corregir esto lo que podemos hacer es declarar nusetra variable nul (dentro de la estructura Post) para permitir que sea nula en ciettos casos.

Con lo anterior si volvemos a correr nuestra app vermeos que ya se ejecuta bien. Ya salen los titulos en forma de lista.

Ahora le agregamos un hstack a la list para agregarle el parametro points.

Pero, ¿y si quisiéramos poder mostrar cada uno de los enlaces dentro de nuestra aplicación Hacker News? Y eso es exactamente lo que haremos en la próxima lección. En la próxima lección, agregaremos los toques finales a nuestra aplicación haciendo que los enlaces se abran dentro de una vista web

Para navegar a otra vista necesitamos agregar un nuevo archivo SwiftUI. Lo llamaré nuestro DetailView porque ese es el detalle que aparecerá cuando hagamos clic en una de las celdas de nuestra lista. Y dentro de este DetailView, lo más importante que se puede pasar será la url como una cadena. Pero, por supuesto, debido a que la URL puede ser nula, voy a configurar esto como una cadena opcional. Y dado que ahora tenemos esta propiedad que debe inicializarse en nuestra vista previa, también tenemos que proporcionar un valor para la propiedad de URL, así que solo voy a poner "https: // www. google. com "

Para darle navegacion:
1. agregamos un navigationLink al HStack en el contentview, lo inicializamos con un destination y un label.
2. En el destination ponemos obvio el detailview() ahi le pasamos el url detailview(url: post.url)
3. En el label le damos enter para que abra unos corchetes (los closure) y metemos dentro el hstack

Ahora tenemos que convertir esta DetailView en una vista web que muestre las noticias reales basadas en esa URL que estaban pasando. Así que vayamos a DetailView e intentemos lograr esto. Ahora, para mostrar una URL dentro de un navegador de manera efectiva sin la barra de búsqueda y las otras partes que hacen que un navegador sea un navegador, de hecho, solo una ventana que muestra básicamente el contenido de una URL, necesitamos usar algo llamado webView.

##Como convertir una vista en una vistaWeb:

Para hacer eso, voy a crear una nueva estructura llamada WebView. Y en esta estructura, voy a intentar construir una versión SwiftUI de un componente UIKit que es WK WebView. Y para usar el WebView, primero debemos recordar importar el WebKit. Entonces, ahora puedo crear mi estructura llamada WebView y se ajustará al protocolo que es UIViewRepresentable. Y esto nos permite crear una SwiftUIView que representa un UIKitView. Entonces presionemos enter en eso y creémoslo. Y ahora puede ver que este WebView no se ajusta al protocolo. Para silenciar estas advertencias, tenemos que crear dos métodos delegados aquí. Uno se llama makeUIView. Y otro se llama updateUIView. Entonces, cuando hagamos esta UIView, intentará crear un UIKit WebView y convertirlo en una SwiftUI View.


## Para convertir una ios app en una app para Mac

1. primero nos vamos al archivo principal
2. Despues hacemos clic en la casilla de verificacion de Mac
3. despues nos vamos a la pestaña signin & capabilities
4. nos logueamos si aun no lo hemos hecho

Y listo
