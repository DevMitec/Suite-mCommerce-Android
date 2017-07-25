# Suite MCommerceMIT Android Versión 1.0

## Prefacio

### ¿Cuál es el proposito de este documento?

Esta guía contiene la información necesaria para que los clientes puedan integrar de manera exitosa la implementación de los servicios Suite _mCommcerceMIT_ para dispositivos móviles **Android**.

## Libreria

Los servicios que proporciona suite mCommerce están contenidos en un archivo aar.

Para la integración se deberá colocar el archivo suitemcommerce.arr en la carpeta app -> lib del proyecto.
Posteriormente se deberán agregar las siguientes dependencias en el archivo build.gradle del proyecto:

* compile(name:'suitemcommerce', ext:'aar')
* compile 'io.card:android-sdk:5.3.2' 


## Inicialización Controlador

Se inicializa el controlador mCommerce para llamar las funciones de la suite. 

`suiteController =  new suiteMCommerce(Environment environment, Context context, SuiteControllerDelegate delegate);`

Donde:
* **environment** es un enum correspondiente a un ambiente (SANDBOX,DEV, QA, PROD)
* **context** es el contexto de la actividad para implementación de funcionalidades.
* **delegate** es la interfaz implementada en el contexto


El activity o contexto que necesite alguno de los servicios deberá implementar la _interfaz_ `SuiteControllerDelegate`.

En caso de que alguno de estos parámetros estén incorrectos, se regresará un error:

* **C002**	Ambiente invalido
* **C003**	Contexto invalido
* **S001**	Error de conexión
* **S002**	Excepción de llave

## Cobro


El cobro se hace mediante WebPay el cual facilita la integración al portal del comercio a través de un HTML enviado por el framework.

Para invocar el servicio de cobro

`suiteController.sndPay(String company, String xmlA, String xmlM); `

Donde:

* **company**: es el ID de la compañía de CDP.
* **xmlA**: es la cadena proporcionada por el integrador
* **xmlM**: es la cadena única que se le proporciona al comercio.
* **(NOTA: Estos datos le serán proporcionados por MIT)**

Para generar la cadena xmlA se debe seguir el siguiente procedimiento:

1. Formar la cadena agregando los datos de la transacción al xml.
2. Encriptar todo el xml a través del algoritmo AES, y con la “semilla” que será proporcionada por MIT, así la cadena será hexadecimal.

Formato de la cadena xmla y los datos que la componen:

<pre>
<code>
&lt;xml&gt;
&lt;tpPago&gt;C&lt;/tpPago&gt;
&lt;amount&gt;0.01&lt;/amount&gt;
&lt;urlResponse&gt;https://suitemcommerce.com&lt;/urlResponse&gt;
&lt;referencia&gt;NUM_FACTURA&lt;/referencia&gt;
&lt;moneda&gt;MXN&lt;/moneda&gt;
&lt;date_hour&gt; 2013-07-10T14:49:24-05:00&lt;date_hour&gt;
</code>
</pre>


**Parametros XMLA

* **tpPago**

Establece el tipo de pago con el que se realizará el cargo, ya sea de contado ó meses. 
<pre><code>
 tpPago           Forma de pago          Tipo de afiliación
C                 Pago de contado        Afiliación de contado 
3M                Pago a 3 meses         Afiliación a 3 meses 
6M                Pago a 6 meses         Afiliación a 6 meses
</code></pre>


**amount**

Establece el Importe por el que se realizará la solicitud de cargo. El importe deberá enviarse sin comas (en caso de miles), con punto y 2 decimales. Ejemplos:
 
<pre><code>
_Valor Incorrecto      Descripción del Error           Valor Correcto_
* 12,530.34            Comas en miles                  12530.34 
* 12.530,34            Formato Europeo                 12530.34 
* 12,530.3476          Más de 2 decimales              12530.34 
* 12 530.34            Espacios en importe             12530.34 
* $-12530.34           Signo negativo y de moneda      12530.34
</code></pre>


**urlResponse**

Está será por default **https://suitemcommerce.com**


**referencia**

Será la referencia por el motivo de cobro


**moneda**

Establece el tipo de moneda con el que se realizará el cargo, los valores que acepta CENTRO DE PAGOS son MXN y USD. 

Los requisitos para el comercio son: 

<pre><code>
 _Moneda                Requisitos_ 
* MXN                   Afiliación Bancaria en Pesos Chequera en Pesos 
* USD                   Afiliación Bancaria en Dólares Chequera en Dólares 
</code></pre>


La regulación mexicana sólo permite el cargo en dólares a clientes del comercio con tarjetas bancarias emitidas fuera de la República Mexicana.


**date_hour**

Fecha/Hora actual del servidor del comercio en formato ISO8601. 
Ejemplo: 
2013-07-10T14:49:24-05:00

**Respuesta**

El delegado que responde a esta función es `didFinishPayProcess(String response, SuiteError suiteError)`

Donde en caso de que la transacción sea exitosa la información de cobro vendrá en el parámetro response y el objeto suiteError vendrá nulo.

> **La respuesta tendrá que ser descifrada con la llave que se encripta el xml.**


El bean tiene los siguientes atributos:


* **Reference**: Referencia del cobro
* **Response**: Aprobada/denegada/error
* **Auth**: Numero de autorización de cobro
* **Error**: Descripción en caso de que la transacción haya sido denegada o errónea
* **ccName**: Nombre del tarjetahabiente
* **ccNum**: Número de la tarjeta
* **Amount**: Monto de la transacción
* **Type**: Tipo de tarjeta

En caso de que haya un error, la respuesta vendrá nulo y el objeto `suiteError` tendrá la información asociada al error.

* **P001**	Compañía invalida
* **P002**	XMLA invalido
* **P003**	XMLM invalido

## Autenticación

Para invocar el servicio de autenticación

`suiteController.authenticate(BeeanTokenization beanTokenization, Bean3DS bean3DS); `

Donde `beeanTokenization` tiene los siguientes atributos:

* **branch** es la sucursal de la empresa
* **company** es el ID de la compañía de CDP
* **country** es el código de pais
* **user** es usuario de CDP.
* **password** es la contraseña asociada al usuario CDP.
* **merchant** es la afiliación
* **currency** es un enum que corresponde al tipo de moneda (MXN, USD)
* **operationType** es el número de operativa para realizar el cobro.
* **reference** es la referencia con la cual se realizará el cobro.


Donde `bean3DS` tiene los siguientes atributos:

* **company** es el ID de la compañía de CDP
* **branch**: es la sucursal de la empresa
* **country**: es el código de pais
* **user**: es usuario de CDP.
* **password**: es la contraseña asociada al usuario CDP.
* **merchant**: es la afiliación
* **currency**: es un enum que corresponde al tipo de moneda (MXN, USD)
* **authKey**: es la llave para procesar la autenticación proporcionada al comercio
* **reference**: es la referencia con la cual se autenticará.
**(NOTA: Estos datos le serán proporcionados por MIT)**


Ejecutará una vista donde se capturarán los datos del tarjetahabiente. 


El delegado que responde a esta función es `didFinishAuthenticationProcess(BeantokenizeResponse beanTokenizeResponse, SuiteError suiteError)`


Donde en caso de que el proceso sea exitoso la información de token vendrá en el `beanTokenizeResponse` y el objeto `suiteError` vendrá nulo. El bean tiene los siguientes atributos:

* **token**: Token
* **nbResponse**: Success o error
* **tokenReference**: Referencia de token

Caso contrario la descripción de porque no se realizó la autenticación vendrá en el objeto `suiteError`

* **A001**	Branch invalido
* **A002**	Country ainvalido
* **A003**	Usuario invalido
* **A004**	Contraseña invalida
* **A005**	Merchant invalido
* **A006**	Moneda invalida
* **A007**	Llave invalida
* **A008**	Parámetros inválidos
* **A009**	Compañía invalida
* **A010**	Autenticación faillida

## Cobro con Token

Para invocar el servicio de cobro

`suiteController.sndPayWithToken(BeeanTokenization beanTokenization, Bean3DS bean3DS); `

Donde `beeanTokenization` tiene los siguientes atributos:

* **branch** es la sucursal de la empresa
* **company** es el ID de la compañía de CDP
* **country** es el código de pais
* **user** es usuario de CDP.
* **password** es la contraseña asociada al usuario CDP.
* **merchant** es la afiliación
* **currency** es un enum que corresponde al tipo de moneda (MXN, USD)
* **operationType** es el número de operativa para realizar el cobro.
* **token** es el token asociado a una tarjeta
* **reference** es la referencia con la cual se realizará el cobro.
* **amount** es el monto por el cual se realizará el cobro.
* **(NOTA: Estos datos le serán proporcionados por MIT)**

Y donde `bean3DS` tiene los siguientes atributos:

* **branch** es la sucursal de la empresa
* **company** es el ID de la compañía de CDP
* **country** es el código de pais
* **user** es usuario de CDP.
* **password** es la contraseña asociada al usuario CDP.
* **merchant** es la afiliación
* **currency** es un enum que corresponde al tipo de moneda (MXN, USD)
* **authKey** es la llave para procesar la autenticación proporcionada al comercio
* **reference** es la referencia con la cual se autenticará.
* **(NOTA: Estos datos le serán proporcionados por MIT)**

El delegado que responde a esta función es `didFinishTokenizeTransaction(BeanPaymentWithToken beanPaymentWithToken, SuiteError suiteError)`

Donde en caso de que el proceso sea exitoso la información del cobro vendrá en el `beanPaymentWithToken` y el objeto `suiteError` vendrá nulo. El bean tiene los siguientes atributos:

* **Reference**: Referencia del cobro
* **Response**: approved/denied/error
* **Auth**: Numero de autorización de cobro
* **Folio**: Folio de la transacción
* **Amount**: Monto de la transacción
* **Date**: Fecha de la transacción
* **Error**: Descripción en caso de que la transacción haya sido denegada o errónea

En caso de que haya un error, el objeto `beanPaymentWithToken` vendrá nulo y el objeto `suiteError` tendrá la información asociada al error.

* **T001**	Branch invalido
* **T002**	Usuario invalido
* **T003**	Contraseña invalida
* **T004**	Tarjeta invalida
* **T005**	País invalido 
* **T006**	Merchant invalido
* **T007**	Referencia invalida
* **T008**	Tipo de operación invalida
* **T009**	Token invalido
* **T010**	Monto invalido
* **T011**	Moneda invalida
* **T012**	Parámetros inválidos
* **T013**	La tarjeta proporcionada no corresponde con el token
* **T014**	Compañía invalida



## Contacto Mesa de Ayuda MIT
Para la atención de los requerimientos de soporte, contamos con mesa de ayuda con los siguientes números en ciudad de México para resolver dudas y orientación para explotar eficientemente los recursos de nuestra plataforma.

Teléfono en la ciudad de México 
**(01.55) 1500.9000**

Correo electrónico 
**soporte.centrodepagos@mitec.com.mx**
