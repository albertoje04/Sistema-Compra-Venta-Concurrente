[![logo](https://www.gnu.org/graphics/gplv3-127x51.png)](https://choosealicense.com/licenses/gpl-3.0/)
# Primera Práctica
Para la resolución de esta primera práctica se utilizará como herramienta de concurrencia los semáforos. Para el análisis y diseño el uso de semáforos es como se ha presentado en las clases de teoría. Para la implementación se hará uso de la clase [`Semaphore`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/Semaphore.html) de JAVA. Hay diferentes ejemplos en este [guión](https://gitlab.com/ssccdd/materialadicional/-/blob/master/README.md) donde se demuestra el uso general de la clase así como soluciones de cursos anteriores. Para la implementación se utilizará la factoría [`Executors`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/Executors.html) y la interface [`ExecutorService`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ExecutorService.html) para la ejecución de las tareas concurrentes que compondrán la solución de la práctica.

## Problema: Sistema de Procesamiento de Pedidos
Vamos a simular un proceso de venta por Internet. Para la simulación deberemos resolver una serie de tareas que deberán ejecutarse de forma concurrente garantizando una máxima eficiencia. 

1.  **RecibirPedido**: Recibe pedidos de los clientes.
2.  **VerificarPago**: Verifica y confirma el pago del pedido.
3.  **ChequearInventario**: Verifica la disponibilidad de los artículos pedidos.
4.  **PrepararPedido**: Empaqueta los artículos una vez que el pago ha sido verificado y los artículos están confirmados como disponibles.
5.  **OrganizarEnvio**: Organiza el envío del paquete al cliente.
6.  **ActualizarInventario**: Actualiza el inventario basado en los artículos enviados.
7.  **NotificarCliente**: Notifica al cliente sobre el estado de su pedido.
8. **EntregarPedido** : El cliente recibe el pedido del transportista y valorará el servicio de envío así como la calidad del producto que ha comprado.

En las diferentes tareas estarán implicados una serie de procesos que deberán utilizar semáforos para garantizar el uso correcto de los datos y la sincronización de las tareas. Los procesos son los siguientes:

 1. proceso **Usuario** : Simula las acciones que deberá seguir el usuario para comprar los articulos que desee en la tienda. Para simplificar el proceso las solicitudes de los artículos se hacen a una tienda que dispone de ellos en su catálogo.
 2. proceso **Tienda** : Tiene un catálogo de productos que desea vender a los clientes.
 3. proceso **Transportista** : Será el encargado de recibir los encargos de transporte de la tienda para enviarlos al cliente correspondiente.
 4. proceso **Proveedor** : Será el encargado de atender las necesidades de inventario de la tienda para completar los productos de su catálogo.

### Para la implementación

Deberán definirse tiempos que simulen las operaciones que tienen que hacer los diferentes procesos implicados en el problema. También deberá definirse un tiempo para la finalización de la prueba. Hay que mostrar información en la consola para seguir la ejecución de los procesos.

El hilo principal deberá crear un número suficiente de procesos para demostrar la concurrencia de los mismo. De todos los procesos deberá haber instancias suficientes.

Además debe implementarse un proceso de finalización que se lanzará pasado un tiempo que estará definido en las constantes. Por tanto todos los procesos presentes en la solución deberán implementar su interrupción como el método de finalización.

## Solución

En este enlace se encontrará el repositorio de GitLab --> [`Repositorio Gitlab`](http://suleiman.ujaen.es:8011/aje00009/jimenez_exposito_alberto_practica_1)

Ahora procedemos a analizar el problema que se nos ha proporcionado para su posterior diseño e implementación

### Análisis

En esta parte se describirán las variables compartidas, estructuras de datos, semáforos y clases auxiliares para la resolución del ejercicio.

### Datos

Tipos de datos (TDA) necesarios para la solución de la práctica:
-   **TDA** `EstadoPedido`
	-   Enumerado (`RECIBIDO`, `PAGADO`,`PREPARADO`,`ENTREGADO`)
    
-   **TDA** `Pedido`
	-   `idPedido : Integer`
	-   `estado : EstadoPedido`
	-   `productos: ArrayList<Producto>`
	-  	`coste: float`
	-   `transporte: Transportista`
	-   `usuario : Usuario` 

-   **TDA** `Producto` 
	-  `idProducto: Integer`
	-  `stock: Integer`
	-  `precio: float`
	-  `mutexValoracion: Semaphore`
	-  `valoracion: float`

-   **TDA** `Almacen` 
	-  `productos: HashMap<Producto>` --> Atributo de `Almacen` que guarda los productos que tiene. Será útil y eficiente utilizar un HashMap dado que el `Usuario` solicitará los productos por su id.
	- operaciones:
		- `find(clave)`: Devuelve un producto si se encuentra en la estructura.
		- `put(clave, Producto)` Inserta un elemento en la estructura

### Variables compartidas

Serán elementos distintos a los que tendrán acceso diferentes clases como son el Usuario, Transportista, Tienda y Proveedor.

Las variables compartidas entre `Usuario` y `Tienda` son las siguientes:

-   `bufferPedidosPendientes : ArrayList<Pedido>`  Almacenará los pedidos hechos por los usuarios y aún no pagados. En el momento en el que se paguen, se pasarán al buffer descrito a continuación. 

Las variables compartidas entre `Tienda` y `Transportista` son las siguientes:
-   `bufferPedidosPagados: ArrayList<Pedido>` Almacenará los pedidos ya pagados por los usuarios que están pendientes de procesar y ser enviados y llevados por los transportistas.

Las variables compartidas entre `Tienda` y `Proveedor` son las siguientes:
- `almacen: Almacen` Será un objeto compartido entre la `Tienda` y el `Proveedor` para acceder a este. Cabe mencionar que su acceso estará debidamente controlado por semáforos.

### Constantes
- `NUM_USUARIOS` Definirá el número total de usuarios que se crearán en el hilo principal
- `NUM_TIENDAS` Número de tiendas totales
- `NUM_PROVEEDORES` Número de proveedores totales
- `NUM_ALMACENES` Número de almacenes totales
- `NUM_TRANSPORTISTAS` Número de transportistas totales
***
- `TAM_BUFFER` Supondremos el mismo tamaño para ambos buffer compartidos descritos anteriormente
- `TAM_PEDIDO_MAX` Número máximo de productos en un `Pedido` 
- `TAM_PEDIDO_MIN` Número mínimo de productos en un `Pedido`
- `MAX_ID` Máximo número  de ID de un producto.
- `MIN_ID` Mínimo número  de ID de un producto
- `STOCK_MAXIMO` Máximas unidades de un producto en un almacén
- `SIN_STOCK` Producto cuyo stock es 0
- `MAX_VALORACION` Máxima puntuación de un `Producto` y/o `Transportista`
***
- `TIEMPO_BUSQUEDA_PRODUCTOS` Tiempo que un `Usuario` tarda en buscar y elegir los productos de su pedido en el catálogo
- `TIEMPO_CREACION_PEDIDO` Tiempo que tarda un `Usuario` en crear un `Pedido`.
- `TIEMPO_PAGO` Tiempo que tarda un `Usuario` en pagar un `Pedido`. 
- `TIEMPO_COGER_PEDIDO_TRANSPORTISTA` Tiempo que el `Transportista` tarda en recoger un pedido de la `Tienda`.
- `TIEMPO_TRANSPORTE` Tiempo del `Transportista` en llevar el paquete hasta el `Usuario` que realizó el pedido.
- `TIEMPO_PROCESAMIENTO_PEDIDO` Tiempo que tarda una `Tienda` en procesar un `Pedido`
- `TIEMPO_REPOSICIÓN` Tiempo que un `Proveedor` tarda en reponer el stock de los productos pertinentes
- `TIEMPO_VALORACION` Tiempo que un `Usuario` tardará en valorar los productos de su `Pedido` y la entrega del `Transportista`.

### Semáforos

En cuanto a los semáforos, utilizaremos algunos para la correcta sincronización de los buffer compartidos (que he supuesto con límite), la sincronización entre `Tienda` y `Proveedor` para el acceso al almacén, etc. No se describirán los semáforos locales a objetos de una clase. Estos han sido nombrados anteriormente y su pseudocódigo se hará explícito a continuación.

##### Semáforos entre `Tienda` y `Usuario`:
- `vaciosPedidosTienda` Representará un semáforo que indicará si el buffer de pedidos realizados por un `Usuario` está vacío, en cuyo caso, la tienda no podrá actuar y se tendrá que bloquear en dicho semáforo. Inicializado a 0.

-   `llenosPedidosTienda` Representa el semáforo "inverso" al anterior. Este nos ayudará a controlar la creación de pedidos por parte del `Usuario` si el buffer ya está lleno y no caben más pedidos. Inicializado a TAM_BUFFER

- `mutexPedidosTienda` Representa el semáforo de acceso en sección crítica del buffer compartido. Inicializado a 1.

- `semPagado` Semáforo que se encontrará dentro de un `Pedido` y nos dirá si se ha pagado o no. Por otro lado, la `Tienda` comprobará hasta que esté pagado. Entonces actuará y procesará el pedido.

##### Semáforos entre `Tienda` y `Transportista`:
-   `vaciosPedidosTransportista` El mismo funcionamiento que el explicado anteriormente. Inicializado a 0.

-  `llenosPedidosTransportista` El mismo funcionamiento que el explicado anteriormente. Inicializado a TAM_BUFFER

- `mutexPedidosTransportista` El mismo funcionamiento que el explicado anteriormente. Inicializado a 1.

##### Semáforos entre `Tienda` y `Proveedor`:
- `mutexAlmacen`  Semáforo que representa el acceso al almacén entre diferentes tiendas de forma segura. Este semáforo asegurará el acceso en exclusión mutua al `Almacen` entre una `Tienda` y un `Proveedor`. Inicializado a 1.

-  `semProveedor` Semáforo que indica la relación entre las tres clases nombradas en este apartado. La tienda despertará a su `Proveedor` cuando necesite reponer stock de algún producto. Inicializado a 0.

-  `semRepuesto` Semáforo  que avisará a la `Tienda` desde el `Proveedor` de que ya se ha repuesto todo lo que fuera necesario, para que la `Tienda` pueda proseguir con su ejecución. Inicializado a 0.

- `semTiendaOcupada` Más bien sería un semáforo que controla el acceso entre varias `Tiendas`, ya que asegura que ninguna tienda podrá entrar a la misma vez que otra. Si no utilizáramos este elemento, al liberar la exclusión mutua del almacén al que está accediendo una `Tienda` para que pueda cogerlo el `Proveedor`, podría darse un error, y que otra tienda entrara a la misma vez. Esto se detallará en la parte de diseño.
    
##### Semáforos entre `Usuario` y `Transportista`:
- `semEntregado` Este semáforo nos permite que el `Transportista` indique al `Usuario` que su pedido ha sido entregado con éxito. Básicamente, una vez realizado el pedido, el `Usuario` esperará en este semáforo hasta que el `Transportista` lo avise. Inicializado a 0.


## Diseño
En esta segunda parte se mostrará una implementación básica con pseudocódigo mostrando los principales método y ejecuciones de las principales clases en nuestro problema. Hay que mencionar que las clases que serán ejecutables serán `Usuario`, `Tienda` , `Transportista` y `Proveedor`.

### Hilo Principal
En el hilo principal se llevará a cabo la creación y ejecución de los procesos.
```
	crearSemáforos(); //Crea los semáforos y los inicializa
	crearVariablesCompartidas(); //Creará variables compartidas
	while (NO FINALIZACION){
		proceso = crearProceso(); //Se crea un proceso
		ejecutarProceso(proceso);
	}
	
	finalizarProcesos(listaProcesos);
```

### Proceso Usuario
Variables locales de la clase:
```
idUsuario: Integer
nombre: String
email: String
pedido: Pedido
direccion: String
dni: String
catalogo: HashMap<Producto>
```

**ejecución()**
```
repeat //Hasta que se interrumpa la ejecución
	crearPedido()
	pagarPedido()
	esperarPedido()
	recibirPedido()
forever
```

*crearPedido()*
El `Usuario` creará un `Pedido` a partir del catálogo que se le proporciona (proveniente de la `Tienda` y a su vez del `Almacen` de la misma). Después se creará el `Pedido` con los producto elegidos, se añadirá a la estructura compartida con los demás usuarios y tiendas, para que una `Tienda` pueda procesarlo. Obviamente, el acceso al buffer se hará en exclusión mutua.
```
	//El usuario elige los productos del catálogo de la Tienda
	ArrayList<Producto> productos = new ArrayList<>();
	int numProductos = aleatorio(TAM_PEDIDO_MIN,TAM_PEDIDO_MAX);
	
	sleep(TIEMPO_BUSQUEDA_PRODUCTOS);
	
	for (int i=0;i<numProductos;i++){
		Producto p = catalogo.find(aleatorio(MIN_ID,MAX_ID));
		productos.add(p);
	}

	sleep(TIEMPO_CREACION_PEDIDO);

	//Luego crea el Pedido
	pedido = new Pedido (productos); //Faltarían los demás atributos de un Pedido
	wait(llenosPedidosTienda);
	wait(mutexPedidosTienda);
	
	bufferPedidosPendientes.add(p); //Sección crítica
	
	signal(mutexPedidosTienda);
	signal(vaciosPedidosTienda);
	
```

*pagarPedido()*
Este método simula simplemente el tiempo que tarda en pagar el `Usuario` un `Pedido`
```
	sleep(TIEMPO_PAGO)
	signal(pedido.semPedido) //Libera el semáforo para despertar a la Tienda
```

*esperarPedido()*
En esta primitiva el `Usuario` esperará a que el transportista entregue el paquete al `Usuario`.
```
	wait(semTransportista) //Espera a que lo despierte el transportista
						//para recibir el pedido
```

*recibirPedido()*
En este método se valorarán los productos y `Transportista` del pedido realizado. Además el `Usuario` recibirá un mensaje de que el paquete ha llegado a su destino.
```
	sleep(TIEMPO_VALORACION);
	
	//Valoracion de productos
	ArrayList<Producto> prod = pedido.getProductos();
	for (Producto p: prod){
		wait(p.mutexValoracion);
		p.addValoracion(aleatorio(MAX_VALORACION));
		signal(p.mutexValoracion);
	}

	//Valoracion del Transportista
	Transportista transporte = pedido.getTransportista();
	wait(transporte.mutexValoracion);
	trasporte.addValoracion(aleatorio(MAX_VALORACION));
	signal(transporte.mutexValoracion);
	

	print("El pedido del cliente "+ idCliente + " ha llegado a su destino)
```

### Proceso Transportista
Variables locales:
```
	idTransportista: Integer
	mutexValoracion: Semaphore
	valoracion: float
	pedido: Pedido
```

**ejecucion()**
```
repeat   //Hasta que se interrumpa
	cogerPedidoListo();
	transportarPedido();
	entregarPedido();
forever
```

*cogerPedidoListo()*
En este método se simula la interacción entre la `Tienda` y el `Transportista` a la hora de coger los pedidos ya listos para su posterior transporte. Se realizará en exclusión mutua. Se asociará el `Transportista` al `Pedido` que ha recogido del buffer.
```
	wait(vaciosPedidosTransportista);
	wait(mutexPedidosTransportista);
	
	sleep(TIEMPO_COGER_PEDIDO_TRANSPORTISTA);
	pedido = bufferPedidosPagados.remove(0);  //Devuelve el primer elemento y lo elimina
	pedido.setTransportista(this);
	
	signal(mutexPedidosTransportista);
	signal(llenosPedidosTransportista);
```

*transportarPedido()*
Simulará el tiempo que tarda un `Transportista` en llevar el paquete hasta la dirección del usuario.
```
	sleep(TIEMPO_TRANSPORTE);
```

*entregarPedido()*
Ahora se procederá a la entrega del pedido del `Usuario` que lo hizo.
```
	signal(semTransportista);   //Despierta al Usuario que estaba esperando
	pedido.setEstado(ENTREGADO);   
```

### Proceso Tienda
Variables locales
```
	idTienda: Integer
	catálogo: HashMap<Producto>
	almacen: Almacen
```

**ejecución()**
```
repeat  //Hasta que se interrumpa su ejecución
	cogerPedido();
	esperarPago();
	procesarPedido();
forever
```

*cogerPedido()*
En este método observamos otro claro ejemplo de productor/consumidor. En especial, la parte opuesta a la expuesta en el `Usuario`.
```
	wait(semVaciosTienda);
	wait(mutexPedidosTienda);
	bufferPedidosPendientes.remove(PRIMERO); //Coge el primer pedido a procesar
	signal(mutexPedidosTienda);
	signal(semLlenosTienda);
```
*esperarPago()*
En este caso, simplemente bloqueamos el proceso `Tienda` hasta que se complete el pago del `Pedido` en proceso.
```
	wait(pedido.semPagado); //Espera a que el pedido que está procesando sea pagado
	pedido.setEstado(PAGADO);
```
*procesarPedido()*
Ahora la `Tienda` procesa el `Pedido` cogiendo los diferentes productos del almacén. Se supondrá que solo se podrá pedir 1 unidad de cada producto. Por otro lado, al terminar, la `Tienda` comprobará si ha dejado algún producto fuera de stock, en cuyo caso, avisará al `Proveedor` para que reponga dichos elementos en el `Almacen`. Por último, pasará el `Pedido` al buffer de pedidos listos para que un `Transportista` pueda recogerlo.
```
	wait(semTiendaOcupada);  //Esperamos a que ninguna Tienda esté procesando un pedido
	wait(mutexAlmacen);  //Exclusión mútua para almacen

	sleep(TIEMPO_PROCESAMIENTO_PEDIDO);
	
	for (Productos prodActual: pedido.getProductos()){
		almacen.cogerProducto(prodActual); //Este metodo rebajará 1 unidad de dicho producto
	} 
	
	if (almacen.fueraDeStock() == true){ //Si hay algún producto fuera de stock
		signal(mutexAlmacen); //Libera el mutex para el proveedor
		signal(semProveedor); //Despierta al proveedor
		wait(semRepuesto); //Espera a que el proveedor reponga
	}	else {
			signal(mutexAlmacen);
		} 
	//Pedido listo, se lo pasamos a los Transportistas
	pedido.setEstado(PREPARADO);
	pasarPedidoTransportista(pedido);
	signal(tiendaOcupada) //Ya hemos terminado la operacion
```

*pasarPedidoTransportista(Pedido p)*
```
	wait(llenosPedidosTransportista);
	wait(mutexPedidosTransportista);
	
	bufferPedidosPagados.add(pedido);
	
	signal(mutexPedidosTransportista);
	signal(vaciosPedidosTransportista);
```

### Proceso Proveedor
Variables locales
```
	idProveedor: Integer
	almacen: Almacen
```

**ejecucion()**
```
	repeat
		reponerStock();
	forever
```

*reponerStock()*
En este método, el `Proveedor` repondrá el stock de los productos fuera de stock mediante la variable compartida con su `Tienda` correspondiente (almacén)
```
	wait(semProveedor);
	wait(mutexAlmacen);

	sleep(TIEMPO_REPOSICION);
	
	HashMap<Producto> prodAlmacen = almacen.getProductos();
	for (Producto p: prodAlmacen){
		if(p.getStock() == SIN_STOCK)
			p.setStock(STOCK_MAXIMO);
	}

	signal(mutexAlmacen);
	signal(semRepuesto);
```

Aquí acabaríamos la parte de diseño de esta práctica. Como podemos ver el sistema se sincroniza gracias al uso de variables compartidas, que serán pasadas por cabecera debidamente en su futura implementación. A su vez, se controlarán gracias a los semáforos mencionados con anterioridad. 

#### `Autor: Alberto Jiménez Expósito`
#### `Correo: aje00009@red.ujaen.es`
#### `Fecha: 10/03/2024`
