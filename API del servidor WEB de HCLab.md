# API del servidor WEB de HCLab

## Readme
A continuación se detallan los endpoints para acceso por API al servidor web de HCLab, pero antes hay que tener en cuenta algunos detalles sobre el esquema de comunicaciones del servidor:
- El esquema de autenticación utlizado para todas las comunicaciones con el servidor es __bearer__, adhiriéndonos al [_framework_ OAuth 2.0](https://www.rfc-editor.org/rfc/rfc6749) en la medida de lo posible. Por lo tanto, todas las solicitudes de recursos deben incluir una cabecera del tipo ```"Authorization": "Bearer " + APItoken```. El token se debe solicitar al endpoint ```api/login```. 
- Las respuestas definidas en esta lista se aplican solo en el caso de que el resultado de la petición sean del rango 200-299, de lo contrario se devolverá:

    ```json
    {
        "status": "error",
        "message": "mensaje de error específico"
    }
    ```
  Con la excepción de las peticiones a api/login que resulten en un código 400 o 403, en cuyo caso se responderá según lo establecido en el [_framework_ OAuth 2.0](https://www.rfc-editor.org/rfc/rfc6749).
- Las fechas en las llamadas a la API se deben introducir en formato ISO8601 (ejemplo: _2024-12-27T10:16:14_). No es necesario incluir horas, minutos y segundos. Si se incluyen, serán descartados.
- Los campos que reciben IDs (por ejemplo cliente u obra) aceptan valores de tipo numérico (tanto _integer_ como _double_) y opcionalmente también strings parseables a número (ejemplo: _"1"_).

## Índice

- [api/login](#apilogin)
- [api/logout](#apilogout)
- [api/menu](#apimenu)
- [api/clientes](#apiclientes)
- [api/obras](#apiobras)
- [api/materiales](#apimateriales)
- [api/suministra](#apisuministra)
- [api/catalogo](#apicatalogo)
- [api/metodos](#apimetodos)
- [api/usuario](#apiusuario)
- [api/ficheros](#apificheros)
- [api/mensajes](#apimensajes)
- [api/actas](#apiactas)
- [api/facturas](#apifacturas)
- [api/amasadas](#apiamasadas)
- [api/ensayos](#apiensayos)
- [api/peticion](#apipeticion)
- [Casos especiales](#casos-especiales)

___
## api/login
**¿Requiere cabecera "Authorization"?** 

No

**Solicitud:** 
```json
{
    "grant_type": "password",
    "username": "usuario",
    "password": "contraseña"
}
```
_*El grant_type debe ser textualmente "password", el username y password deben ser los credenciales del usuario del servidor web de HCLab._

**Respuesta:**
```json
{
    "access_token": "Bearer {token de acceso}",
    "token_type": "bearer (esta API no utiliza MAC, siempre bearer)",
    "expires_in": "caducidad en segundos"
}
```
**Consideraciones importantes:** 

La caducidad del token se resetea a **15min** con cada petición exitosa que utiliza dicho token hasta alcanzar una duración máxima de **120min**.

Como se aprecia en la respuesta de ejemplo, no aceptamos el uso de __Refresh tokens__. Se debe volver a iniciar sesión cuando el __access token__ expira.

El __scope__ por defecto de esta API será aquel definido por el laboratorio en HCLab. A día de hoy la API no acepta solicitudes con un __scope__ restringido.

___
## api/logout
Cierra sesión, borrando el token de sesión API.

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
vacía

**Respuesta:**
```json
{
    "status": "success",
    "message": "You have been logged out successfully",
}
```

___
## api/menu

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
vacía

**Respuesta:**

Devuelve una serie de opciones a las que tiene acceso el usuario autenticado. 

Ejemplo:

```json
[
  {
    "titulo": "Probetas",
    "url": "/amasadas",
    "hint": "Concrete samples"
  },
  {
    "titulo": "Facturas",
    "url": "/facturas",
    "hint": "Invoices"
  },
  {
    "titulo": "Peticiones",
    "url": "/peticion",
    "hint": "Visit request"
  }
]
```

___
## api/clientes
Devuelve la lista de __clientes__ a los que se tiene acceso según opción (actas, facturas, probetas o amasadas).

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
Parametros para valores posibles: actas, facturas, probetas o amasadas.

```json
  {
    "para": "facturas"
  }
```

**Respuesta:**
```json
[
  {
    "name": "Antonio Perez Casas",
    "id": "1"
  },
  {
    "name": "Dragados",
    "id": "3"
  },
  {
    "name": "Ferrovial",
    "id": "7"
  },
  {
    "name": "Geconcesa",
    "id": "6"
  },
  {
    "name": "OHL",
    "id": "2"
  },
  {
    "name": "Sacyr",
    "id": "4"
  }
]
```

___
## api/obras
Devuelve la lista de __obras__ a los que se tiene acceso según opción (actas, facturas, probetas o amasadas).

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
Parametros para valores posibles: actas, facturas, probetas, amasadas o peticion.

```json
  {
    "para": "actas"
  }
```

**Respuesta:**
```json
[
  {
    "name": "10: Torre Mayor, 8 plantas, Calle Mayor, Altea",
    "id": "10"
  },
  {
    "name": "12: Aparcamiento 3 plantas, Avenida de la Libertad, Murcia",
    "id": "12"
  },
  {
    "name": "14: Fábrica de colchones, Poligono la Completa, 109, Altea",
    "id": "14"
  },
  {
    "name": "16: Edificio Altavista, Calle Juan Sebastian Elcano, 12, Murcia",
    "id": "16"
  },
  {
    "name": "1: Vivienda Unifamiliar, C/ Fuensanta 3, Murcia",
    "id": "1"
  },
  {
    "name": "2: Duplex en urbanización Camposol, C/Olivos, 23, Alicante",
    "id": "2"
  },
  {
    "name": "3: Puerto deportivo del sureste, Paseo deportivo de Altea, 2, Altea",
    "id": "3"
  },
  {
    "name": "4: Nave 1, Calle Buenos Aires, 15, Alicante",
    "id": "4"
  },
  {
    "name": "5: Vivienda de 2 plantas, Calle Acuario, 2, Murcia",
    "id": "5"
  },
  {
    "name": "7: Vivienda de 3 plantas, C/ Los Álamos, 8, Espinardo",
    "id": "7"
  }
]
```

___
## api/materiales
Devuelve la lista de __materiales__ contenidos en actas a las que se tiene acceso, para poder filtrar de entre las actas disponibles aquellas que utilicen cierto material.
**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "name": "Acero",
    "id": "2"
  },
  {
    "name": "Hormigón",
    "id": "3"
  },
  {
    "name": "Instalación",
    "id": "5"
  },
  {
    "name": "Papel gramaje 120",
    "id": "6"
  },
  {
    "name": "Sondeo",
    "id": "9"
  },
  {
    "name": "Suelo",
    "id": "4"
  },
  {
    "name": "Zahorra artificial",
    "id": "1"
  }
]
```

___
## api/suministra
Devuelve la lista de __suministradores de hormigón__ que figuran en amasadas a las que se tiene acceso, para poder filtrar de entre las amasadas disponibles aquellas que provengan de cierto suministrador.

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "name": "Hormisa, Planta Levante",
    "id": "P2"
  },
  {
    "name": "Hormisa, Planta Sur",
    "id": "P1"
  }
]
```

___
## api/catalogo
Devuelve la lista de __ensayos de catálogo__ usados en las obras a las que tienen acceso.

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "name": "CBR: CBR completo a tres moldes con indice al 95% y 98%",
    "id": "CBR"
  },
  {
    "name": "DENSIDAD: Densidad in situ por el método radiactivo",
    "id": "DENSIDAD"
  },
  {
    "name": "EDOM-1: Ensayo edométrico",
    "id": "EDOM-1"
  },
  {
    "name": "EST-001: Ensayo de estanqueidad de cubiertas",
    "id": "EST-001"
  },
  {
    "name": "GRAN-SU: Ensayo de granulometría y límites",
    "id": "GRAN-SU"
  },
  {
    "name": "HR3: Toma de hormigón de 3 probetas",
    "id": "HR3"
  },
  {
    "name": "IBERTES: Acero HC Ibertest",
    "id": "IBERTES"
  },
  {
    "name": "PENET-1: Ensayo de penetración dinámica DPSH/Borros",
    "id": "PENET-1"
  },
  {
    "name": "PLACA: Placa de carga",
    "id": "PLACA"
  },
  {
    "name": "PROCTOR: Proctor modificado",
    "id": "PROCTOR"
  },
  {
    "name": "SONDEO-1: Ensayo de calicata",
    "id": "SONDEO-1"
  },
  {
    "name": "ZMAQ: Posicionamiento de maquinaria en el lugar del sondeo",
    "id": "ZMAQ"
  }
]
```

___
## api/metodos
Devuelve la lista de __metodos de vertido del hormigón__ que figuran en amasadas a las que se tiene acceso, para poder filtrar de entre las amasadas disponibles aquellas que se hayan vertido usando un determinado método.

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "texto": "Camión"
  }
]
```

___
## api/usuario
Devuelve los datos del usuario web actual.

**¿Requiere cabecera "Authorization"?** 

Sí

**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "user": "WEB",
    "movil": "666554433",
    "email": "usuarioweb@hcsoft.net",
    "descripcion": "Usuario de demostración"
  }
]
```

___
## api/ficheros
Integra varias acciones sobre los ficheros disponibles en la carpeta en el servidor web del usuario que realice la solicitud. 

**¿Requiere cabecera "Authorization"?** 

Sí

Se pueden realizar las siguientes acciones:

### Acción: Consulta
Si la solicitud está vacía devuleve una lista de los ficheros disponibles. 
- __name__: Nombre del fichero, con extensión.
- __path__: Ruta al fichero dentro del servidor. Donde figura 'usr' hace referencia al usuario con el que se realizó el login.
- __message__: Si existe un fichero con el mismo "name" pero con extensión ".info", se lee su contenido y se vuelca aquí. Importante: el nombre del fichero debe incluir la extensión de aquel al que hace referencia, en este ejemplo sería "actafirmada.pdf.info".
- __size__: Tamaño del fichero.
- __date__: Fecha de creación del fichero.
- __removable__: booleano indicando si el usuario tiene permisos para borrar el fichero.

**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "name": "actafirmada.pdf",
    "path": "ficheros/usr/actafirmada.pdf",
    "message": "Ejemplo de acta firmada.<br>",
    "size": "334 KB",
    "date": "26/12/2023",
    "removable": "TRUE"
  }
]
```

### Acción: Descarga
Para **descargar** uno de los archivos mostrados en la consulta anterior, se lanza la petición a ```api/ficheros/usr/\<filename>``` indicando en el json de la solicitud ```"action": "download"``` y la api devuelve el fichero.

**Solicitud:** api/ficheros/usr/\<filename>
```json
{
  "action": "download"
}
```

**Respuesta:**
```json
fichero solicitado
```

### Acción: Borrado
Para **borrar** uno de los archivos mostrados en la consulta anterior, se lanza la petición a ```api/ficheros/usr/\<filename>``` indicando en el json de la solicitud ```"action": "delete"``` y la api confirma el borrado. Según la configuración del servidor el borrado puede cumplir la función de ocultar el fichero en futuras consultas de ficheros disponibles, o borrarlo totalmente.

**Solicitud:**
```json
{
  "action": "delete"
}
```

**Respuesta:**
```json
[
  {
    "result": "TRUE",
    "zona": "F"
  }
]
```

___
## api/mensajes
Integra varias acciones sobre los mensajes disponibles en la carpeta en el servidor web del usuario que realice la solicitud. 

**¿Requiere cabecera "Authorization"?** 

Sí

Se pueden realizar las siguientes acciones:

### Acción: Consulta
Si la solicitud está vacía devuleve una lista de los mensajes disponibles que no tengan la marca de .visto. 
- __name__: Nombre del archivo del mensaje, con extensión.
- __path__: Ruta al archivo del mensaje dentro del servidor.
- __message__: El contenido del archivo del mensaje.
- __removable__: booleano indicando si el usuario tiene permisos para borrar el fichero. De no tenerlos, la opción de borrado solo le añadirá la marca de ".visto" de forma que no volverá a aparecer en futuras consultas.

**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "name": "mensajePrueba.txt",
    "path": "mensajes/usr/mensajePrueba.txt",
    "message": "Este es un mensaje de prueba para la API del srvWeb<br>",
    "removable": "TRUE"
  }
]
```

### Acción: Descarga
Para **descargar** uno de los archivos de mensaje mostrados en la consulta anterior, se lanza la petición a ```api/mensajes/usr/\<filename>``` indicando en el json de la solicitud ```"action": "download"``` y la api devuelve el archivo del mensaje.

**Solicitud:** api/ficheros/usr/\<filename>
```json
{
  "action": "download"
}
```

**Respuesta:**
```text
fichero solicitado (content-type: text/plain)
```

### Acción: Borrado
Para **borrar** uno de los archivos mostrados en la consulta anterior, se lanza la petición a ```api/ficheros/usr/\<filename>``` indicando en el json de la solicitud ```"action": "delete"``` y la api confirma el borrado. Según la configuración del servidor el borrado puede cumplir la función de ocultar el fichero en futuras consultas de ficheros disponibles, o borrarlo totalmente.

**Solicitud:**
```json
{
  "action": "delete"
}
```

**Respuesta:**
```json
[
  {
    "result": "TRUE",
    "zona": "M"
  }
]
```

___
## api/actas

Integra varias acciones sobre las **actas** disponibles en la carpeta en el servidor web del usuario que realice la solicitud.

#### Acción: Consulta

Permite consultar las **actas** emitidas a las que el usuario tiene acceso. Se pueden especificar una serie de filtros para afinar la consulta, o dejarlos en blanco para obtener todas las actas.

**¿Requiere cabecera "Authorization"?** 

Sí

**Parametros de filtro**
```json
{
  "Cliente": "",
  "Obra": "",
  "FechaIni": "",
  "FechaFin": "",
  "FecMueIni": "",
  "FecMueFin": "",
  "Albaran": "",
  "DescMat": "",
  "SuRef": "",
  "RecogeEn": "",
  "Localiza": "",
  "Material": "",
  "Suministra": "",
  "Catalogo": ""
}
```
**Parámetros de respuesta**
- __pdf__: ruta y nombre del pdf del acta, con extensión. Una solicitud vacía contra esta URL permite descargar el acta en pdf (siempre con token de autenticación).
- __fechaimpre__: fecha de emisión del acta.
- __numacta__: número del acta.
- __muestra__: número de muestra ensayada en el acta.
- __obra__: número de la obra a la que pertenece el acta.
- __nombre__: nombre de la obra.
- __impresa__: número que indica si el acta ha sido emitida (0=no, 1=sí).
- __ensayos__: códigos de catálogo de los ensayos contenidos en el acta.
- __ncli__: número del cliente.
- __cliente__: nombre del cliente.
- __fechamuestreo__: fecha en la que se realizó la toma de la muestra ensayada en el acta.
- __instalacion__: número de la instalación a la que pertenece el acta.
- __apto__: En los casos en los que exista una valoración de "apto" o "no apto" indica si todos los resultados del acta son positivos ("Positiva") o si existe alguno negativo ("Negativa"). En los casos en los que no se aplique, devuelve "---".


#### Caso 1: Sin filtro -> Todas las actas a las que se tiene acceso
**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "pdf": "api/actas/9.pdf",
    "fechaimpre": "2022-03-23",
    "numacta": 4,
    "muestra": 5,
    "obra": 3,
    "nombre": "Puerto deportivo del sureste, Paseo deportivo de Altea, 2, Altea",
    "impresa": 1,
    "ensayos": "EST-001",
    "ncli": 2,
    "cliente": "OHL",
    "fechamuestreo": "2022-03-23",
    "instalacion": "",
    "apto": "Negativa"
  },
  {
    "pdf": "api/actas/22.pdf",
    "fechaimpre": "2022-10-03",
    "numacta": 14,
    "muestra": 28,
    "obra": 10,
    "nombre": "Torre Mayor, 8 plantas, Calle Mayor, Altea",
    "impresa": 1,
    "ensayos": "IBERTES",
    "ncli": 2,
    "cliente": "OHL",
    "fechamuestreo": "2022-09-30",
    "instalacion": "",
    "apto": "---"
  },
  {
    "pdf": "api/actas/55.pdf",
    "fechaimpre": "2023-07-24",
    "numacta": 14,
    "muestra": 29,
    "obra": 10,
    "nombre": "Torre Mayor, 8 plantas, Calle Mayor, Altea",
    "impresa": 1,
    "ensayos": "IBERTES",
    "ncli": 2,
    "cliente": "OHL",
    "fechamuestreo": "2022-10-03",
    "instalacion": "",
    "apto": "---"
  }
]
```

#### Caso 2: Filtrando -> Actas a las que se tiene acceso y que cumplen el filtro

**Solicitud:**

```json
{
  "Obra": 10,
  "FechaIni": "2022-01-01",
  "FechaFin": "2022-12-31",
}
```

**Respuesta:**
```json
{
  "pdf": "api/actas/22.pdf",
  "fechaimpre": "2022-10-03",
  "numacta": 14,
  "muestra": 28,
  "obra": 10,
  "nombre": "Torre Mayor, 8 plantas, Calle Mayor, Altea",
  "impresa": 1,
  "ensayos": "IBERTES",
  "ncli": 2,
  "cliente": "OHL",
  "fechamuestreo": "2022-09-30",
  "instalacion": "",
  "apto": "---"
}
```

### Acción: Descarga

Permite descargar una acta concreta identificada por el nombre del archivo con su extensión (basta con copiar el contenido del nodo "pdf" de la consulta anterior).

**Solicitud:** api/actas/<nombreActa>.pdf

**Respuesta:** fichero pdf.


___
## api/facturas

Integra varias acciones sobre las **facturas** disponibles en la carpeta en el servidor web del usuario que realice la solicitud.

### Acción: Consulta

Permite consultar las **facturas** emitidas a las que el usuario tiene acceso según la configuración del laboratorio. Se pueden especificar una serie de filtros para afinar la consulta, o dejarlos en blanco para obtener todas las facturas.

**¿Requiere cabecera "Authorization"?** 

Sí

**Parametros de filtro**
```json
{
  "Cliente": "",
  "Obra": "",
  "FechaIni": "",
  "FechaFin": "",
  "PorPagar": false //Tipo booleano
}
```
**Parámetros de respuesta**
- __pdf__: ruta y nombre del pdf de la factura, con extensión. Una solicitud vacía contra esta URL permite descargar la factura en pdf (siempre con token de autenticación).
- __serie__: serie de facturación.
- __factura__: número de factura
- __fecha__: fecha de emisión de la factura.
- __obra__: número de la obra a la que pertenece la factura.
- __expediente__: número de expediente a la que pertenece la factura.
- __nomobra__: nombre de la obra.
- __direccion__: dirección de la obra.
- __poblacion__: población de la obra.
- __cliente__: número del cliente.
- __nomcliente__: nombre del cliente.
- __base__: base imponible de la factura.
- __tipoiva__: Tipo de IVA de la factura, indicado en porcentaje (ejemplo:_21_).
- __iva__: Resultado de aplicar el tipo de IVA a la base.
- __total__: Suma de base e IVA.
- __nomfpago__: Forma de pago.
- __fechavence__: Fecha de vencimiento.
- __tipoobra__: .
- __origen__: Indica si la factura es a origen (0=no, 1=sí).
- __tarifa__: ID de la tarifa de la factura (interno del laboratorio).
- __agente__: Nombre del agente de cobro asociado a la factura.
- __nomcentro__: Nombre del centro de trabajo que genera la factura.
- __proforma__: Indica si la factura es de tipo proforma (0=no, 1=sí). 


#### Caso 1: Sin filtro -> Todas las actas a las que se tiene acceso
**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "pdf": "",
    "serie": "A",
    "factura": 6,
    "fecha": "2022-12-15",
    "obra": 4,
    "expediente": 4,
    "nomobra": "Nave 1, Calle Buenos Aires, 15, Alicante",
    "direccion": "Calle Buenos Aires, 15",
    "poblacion": "Alicante",
    "cliente": 2,
    "nomcliente": "OHL",
    "base": 660,
    "tipoiva": 21,
    "iva": 138.6,
    "total": 798.6,
    "nomfpago": "Transferencia",
    "fechavence": "2022-12-25",
    "tipoobra": "",
    "origen": 0,
    "tarifa": 1,
    "agente": "Juanjo Martinez Lorca",
    "nomcentro": "Tomelloso",
    "proforma": 0
  },
  {
    "pdf": "",
    "serie": "P",
    "factura": 4,
    "fecha": "2022-09-20",
    "obra": 4,
    "expediente": 4,
    "nomobra": "Nave 1, Calle Buenos Aires, 15, Alicante",
    "direccion": "Calle Buenos Aires, 15",
    "poblacion": "Alicante",
    "cliente": 2,
    "nomcliente": "OHL",
    "base": 0,
    "tipoiva": 16,
    "iva": 0,
    "total": 0,
    "nomfpago": "Transferencia",
    "fechavence": "2022-09-30",
    "tipoobra": "",
    "origen": 0,
    "tarifa": 1,
    "agente": "Juanjo Martinez Lorca",
    "nomcentro": "Tomelloso",
    "proforma": 1
  },
  {
    "pdf": "api/facturas/21.pdf",
    "serie": "A",
    "factura": 12,
    "fecha": "2023-08-21",
    "obra": 4,
    "expediente": 4,
    "nomobra": "Nave 1, Calle Buenos Aires, 15, Alicante",
    "direccion": "Calle Buenos Aires, 15",
    "poblacion": "Alicante",
    "cliente": 2,
    "nomcliente": "OHL",
    "base": 150,
    "tipoiva": 21,
    "iva": 31.5,
    "total": 181.5,
    "nomfpago": "Transferencia",
    "fechavence": "2023-08-31",
    "tipoobra": "",
    "origen": 0,
    "tarifa": 1,
    "agente": "Juanjo Martinez Lorca",
    "nomcentro": "Tomelloso",
    "proforma": 0
  }
]
```

#### Caso 2: Filtrando -> Actas a las que se tiene acceso y que cumplen el filtro

**Solicitud:**

```json
{
  "Obra": 4,
  "FechaIni": "2023-01-01",
  "FechaFin": "2023-12-31",
}
```

**Respuesta:**
```json
{
  "pdf": "api/facturas/21.pdf",
  "serie": "A",
  "factura": 12,
  "fecha": "2023-08-21",
  "obra": 4,
  "expediente": 4,
  "nomobra": "Nave 1, Calle Buenos Aires, 15, Alicante",
  "direccion": "Calle Buenos Aires, 15",
  "poblacion": "Alicante",
  "cliente": 2,
  "nomcliente": "OHL",
  "base": 150,
  "tipoiva": 21,
  "iva": 31.5,
  "total": 181.5,
  "nomfpago": "Transferencia",
  "fechavence": "2023-08-31",
  "tipoobra": "",
  "origen": 0,
  "tarifa": 1,
  "agente": "Juanjo Martinez Lorca",
  "nomcentro": "Tomelloso",
  "proforma": 0
}
```

### Acción: Descarga

Permite descargar una factura concreta identificada por el nombre del archivo con su extensión (basta con copiar el contenido del nodo "pdf" de la consulta anterior).

**Solicitud:** api/facturas/<nombreFactura>.pdf

**Respuesta:** fichero pdf

____
## api/amasadas
Permite consultar las **amasadas** a los que el usuario tiene acceso. Se pueden especificar una serie de filtros para afinar la consulta, o dejarlos en blanco para obtener todos los ensayos.

**¿Requiere cabecera "Authorization"?** 

Sí


**Parámetros de soliocitud**
```json
{
  "Cliente": "",
  "Obra": "",
  "Designación": "",  
  "Localiza": "",
  "Ultimas": "",
  "RealizaIni": "",
  "RealizaFin": "",
  "RompeIni": "",
  "RompeFin": ""
}
```

**Parámetros de respuesta**
- __fechafab__: fecha de fabricación de la amasada.
- __idensayo__: número identificador del ensayo.
- __ensayo__: código de catálogo del ensayo de hormigón utilizado.
- __libro__: código del libro de registro al que pertenece la amasada.
- __muestra__: número identificador de la muestra.
- __albaran__: código de albarán interno de la amasada.
- __sualbaran__: código de albarán asignado por el cliente.
- __lote__: nombre del lote al que pertenece la amasada.
- __nomcapitulo__: nombre del capítulo al que pertenece la amasada.
- __nomrecoge__: nombre del laborante que realizó la recogida de las probetas.
- __obra__: número de la obra al que pertenece la amasada.
- __expediente__: número del expediente al que pertenece la amasada.
- __nomobra__: nombre de la obra.
- __departamento__: número identificador del departamento encargado de la realización del ensayo.
- __designacion__: designación del hormigón utilizado en la amasada.
- __localización__: localización de la amasada.
- __matricula__: matrícula del camión suministrador del hormigón.
- __tipocemento__: tipo de cemento utilizado.
- __probetas__: números identificativos de las probetas de la amasada.
- __horatoma__: hora de realización de la toma escrita en 4 dígitos. Los dos primeros indican horas y los dos últimos minutos.
- __loteletra__: letra identificadora del lote.
- __aguacemento__: relación agua / cemento de la amasada.
- __conom__: resultado del cono medio.
- __ntoma__: número de toma.


#### Caso 1: Sin filtro -> Todas las amasadas a las que se tiene acceso
**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "fechafab": "2024-07-10",
    "idensayo": 145,
    "ensayo": "HR3",
    "libro": 1,
    "muestra": 9,
    "albaran": "T109-1",
    "sualbaran": "",
    "lote": "Pilares S2",
    "nomcapitulo": "Hormigones",
    "nomrecoge": "Miguel Hernández Madrigal",
    "obra": 16,
    "expediente": 11,
    "nombobra": "Edificio Altavista, Calle Juan Sebastian Elcano, 12, Murcia",
    "departamento": 2,
    "designacion": "HA-25/P/20/IIa",
    "localizacion": "Pilares S2",
    "matricula": "",
    "tipocemento": "",
    "probetas": "190, 191, 192, 193, 194",
    "horatoma": 1405,
    "loteletra": "1",
    "aguacemento": "",
    "conom": 0,
    "ntoma": 1
  },
  {
    "fechafab": "2024-10-15",
    "idensayo": 163,
    "ensayo": "HR3",
    "libro": 1,
    "muestra": 15,
    "albaran": "",
    "sualbaran": "",
    "lote": "Pilares -2 y -1",
    "nomcapitulo": "Hormigón",
    "nomrecoge": "Miguel Hernández Madrigal",
    "obra": 13,
    "expediente": 8,
    "nombobra": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornellá de Llobregat",
    "departamento": 2,
    "designacion": "HA-25/P/20/IIa",
    "localizacion": "",
    "matricula": "MU1917BL",
    "tipocemento": "",
    "probetas": "196, 195, 197, 198, 199",
    "horatoma": 1000,
    "loteletra": "",
    "aguacemento": "3",
    "conom": 1,
    "ntoma": 1
  }
]
```

#### Caso 2: Filtrando -> Ensayos a los que se tiene acceso y que cumplen el filtro

**Solicitud:**
```json
{
  "Obra": "16",
  "RealizaIni": "2024-10-01",
  "RealizaFin": "2024-10-30",
}
```

**Respuesta**
```json
[
  {
    "fechafab": "2024-10-15",
    "idensayo": 163,
    "ensayo": "HR3",
    "libro": 1,
    "muestra": 15,
    "albaran": "",
    "sualbaran": "",
    "lote": "Pilares -2 y -1",
    "nomcapitulo": "Hormigón",
    "nomrecoge": "Miguel Hernández Madrigal",
    "obra": 13,
    "expediente": 8,
    "nombobra": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornellá de Llobregat",
    "departamento": 2,
    "designacion": "HA-25/P/20/IIa",
    "localizacion": "",
    "matricula": "MU1917BL",
    "tipocemento": "",
    "probetas": "196, 195, 197, 198, 199",
    "horatoma": 1000,
    "loteletra": "",
    "aguacemento": "3",
    "conom": 1,
    "ntoma": 1
  }
]
```
____
## api/ensayos

Permite consultar los **ensayos** a los que el usuario tiene acceso. Se pueden especificar una serie de filtros para afinar la consulta, o dejarlos en blanco para obtener todos los ensayos.

**¿Requiere cabecera "Authorization"?** 

Sí


**Parámetros de soliocitud**
```json
{
  "Cliente": "",
  "Obra": "",
  "FecMueIni": "",
  "FecMueFin": "",
  "Albaran": "",
  "SuRef": "",
  "RecogeEn": "",
  "Catalogo": ""
}
```

**Parámetros de respuesta**
- __idensayo__: número identificador del ensayo.
- __nomensayo__: descripción corta del ensayo.
- __udes_presup__: unidades presupuestadas.
- __precio__: precio unitario del ensayo.
- __importe__: importe facturado.
- __cota__: en sondeos, indica la cota de la muestra ensayada.
- __udes.factura__: unidades facturadas.


#### Caso 1: Sin filtro -> Todos los ensayos a las que se tiene acceso
**Solicitud:**
vacía

**Respuesta:**
```json
[
  {
    "idensayo": 13,
    "nomensayo": "Toma de hormigón de 3 probetas, EHE-08",
    "udes_presup": 1,
    "precio": 120,
    "importe": 120,
    "cota": "",
    "udes.factura": "1.00+2.00"
  },
  {
    "idensayo": 14,
    "nomensayo": "Toma de hormigón de 3 probetas, EHE-08",
    "udes_presup": 2,
    "precio": 120,
    "importe": 120,
    "cota": "",
    "udes.factura": "1.00+0.00"
  },
  {
    "idensayo": 15,
    "nomensayo": "Acero HC Ibertest, UNE 36608-36065",
    "udes_presup": 10,
    "precio": 150,
    "importe": 150,
    "cota": "",
    "udes.factura": "1.00+0.00"
  }
]
```

#### Caso 2: Filtrando -> Ensayos a los que se tiene acceso y que cumplen el filtro

**Solicitud:**

```json
{
  "Obra": 10,
  "FechaMueIni": "2022-01-01",
  "FechaMueFin": "2022-12-31",
}
```

**Respuesta:**
```json
{
  "idensayo": 15,
  "nomensayo": "Acero HC Ibertest, UNE 36608-36065",
  "udes_presup": 10,
  "precio": 150,
  "importe": 150,
  "cota": "",
  "udes.factura": "1.00+0.00"
}
```


____
## api/peticion

Permite consultar las peticiones de visita a obra realizadas. Se pueden especificar una serie de filtros para indicar obra, rango de fechas o estado de la solicitud. Además se pueden realizar acciones específicas de alta o cancelación de una petición concreta identificada por su id.

**¿Requiere cabecera "Authorization"?** 

Sí

**Parámetros de soliocitud**
```json
{
	"obra": "",
	"situa": "", // A continuación se detallan los valores aceptados
	"visitaFin": "",
	"visitaIni": ""
}
```

**Valores aceptados para "situa"**
- "P": pendientes de aceptar.
- "A": aceptadas.
- "X": pendientes y aceptadas.
- "C": canceladas por el peticionario.
- "R": rechazadas por el laboratorio.
- "T": terminadas o realizadas.

**Parámetros de respuesta**
- __idpeticion__: número identificador de la petición.
- __idobra__: número identificador de la obra.
- __nombre__: nombre de la obra.
- __alta__: fecha de alta de la petición.
- __fecha__: fecha de la visita.
- __horainicio__: inicio del rango de horas para realizar la visita.
- __horafin__: fin del rango de horas para realizar la visita.
- __notas__: notas asociadas a la petición.
- __estado__: estado de la petición (corresponde al desglose de "situa" arriba).
- __comenta__: comentario del laboratorio sobre la petición. Suele incluirse especialmente cuando se rechaza una petición para proponer un cambio de hora o fecha.
- __tipo__: indica el trabajo asociado a la visita ("A" amasada, "R" recogida de muestra o "I" ensayo in situ).
- __tomas__: indica el número de tomas a realizar en la visita.
- __volumen__: indica el volumen de hormigón a ensayar.
- __cantidad__: indica la cantidad de muestras que se deben recoger.
- __material__: indica el material de la muestra a recoger.
- __ensayos__: indica los ids de los ensayos a realizar (por orden correlativo del presupuesto).
- __capitulo__: indica el id del capítulo del presupuesto al que pertenecen los ensayos a realizar.
- __vertido__: indica la localización del elemento hormigonado del que se va a realizar la toma de probetas.
- __metodo__: indica el método de vertido del hormigón.
- __suministrador__: indica el suministrador del hormigón.
- __modifica__: indica la fecha de última modificación de la petición.
- __estadovisita__: indica el estado de la visita asociada a la petición, solo se aplica cuando el estado de la petición es "A" (aceptado) y puede utilizarse para reflejar incidencias durante la visita.

#### Caso 1: Sin filtro -> Todas las peticiones a las que se tiene acceso
**Solicitud:**
vacía

**Respuesta:**
```json
[
	{
		"idpeticion": "9",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T11:36",
		"fecha": "2025-05-28",
		"horainicio": "1400",
		"horafin": "1500",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "A",
		"tomas": "1",
		"volumen": "1",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "Losa",
		"metodo": "Cami&oacute;n",
		"suministrador": "Hormisa Sur",
		"modifica": "2025-05-26T11:36",
		"estadovisita": ""
	},
	{
		"idpeticion": "16",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T13:56",
		"fecha": "2025-05-27",
		"horainicio": "1200",
		"horafin": "1400",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "I",
		"tomas": "0",
		"volumen": "0",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2025-05-26T13:56",
		"estadovisita": ""
	},
	{
		"idpeticion": "11",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T12:29",
		"fecha": "2025-05-27",
		"horainicio": "1200",
		"horafin": "1300",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "A",
		"tomas": "1",
		"volumen": "1",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "Losa",
		"metodo": "Cami&oacute;n",
		"suministrador": "Hormig&oacute;n del sur",
		"modifica": "2025-05-26T12:29",
		"estadovisita": ""
	},
	{
		"idpeticion": "8",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T09:41",
		"fecha": "2025-05-27",
		"horainicio": "1000",
		"horafin": "1200",
		"notas": "Que venga Rita, por favor.",
		"estado": "R",
		"comenta": "Ese d&iacute;a a esa hora no tenemos laborantes disponibles para realizar la visita. &iquest;Podr&iacute;amos aplazarla a media tarde?",
		"tipo": "A",
		"tomas": "1",
		"volumen": "1",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "encofrado",
		"metodo": "Cami&oacute;n",
		"suministrador": "Hormisa",
		"modifica": "2025-05-29T10:50",
		"estadovisita": ""
	},
	{
		"idpeticion": "15",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T13:56",
		"fecha": "2025-05-27",
		"horainicio": "645",
		"horafin": "1000",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "R",
		"tomas": "0",
		"volumen": "0",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2025-05-26T13:56",
		"estadovisita": ""
	},
	{
		"idpeticion": "7",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-03-03T13:58",
		"fecha": "2025-03-04",
		"horainicio": "1100",
		"horafin": "0",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "A",
		"tomas": "1",
		"volumen": "1",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2025-03-03T13:58",
		"estadovisita": ""
	},
	{
		"idpeticion": "4",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2023-07-31T13:48",
		"fecha": "2023-08-01",
		"horainicio": "1100",
		"horafin": "1200",
		"notas": "",
		"estado": "A",
		"comenta": "",
		"tipo": "R",
		"tomas": "0",
		"volumen": "0",
		"cantidad": "5",
		"material": "Aceros",
		"ensayos": "",
		"capitulo": "",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2023-07-31T13:49",
		"estadovisita": " "
	},
	{
		"idpeticion": "1",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2023-07-31T13:30",
		"fecha": "2023-08-01",
		"horainicio": "1100",
		"horafin": "1400",
		"notas": "Va a hacer calor, traer gorra",
		"estado": "A",
		"comenta": "",
		"tipo": "I",
		"tomas": "0",
		"volumen": "0",
		"cantidad": "1",
		"material": "",
		"ensayos": "12",
		"capitulo": "1",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2023-07-31T13:39",
		"estadovisita": " "
	},
	{
		"idpeticion": "3",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2023-07-31T13:45",
		"fecha": "2023-08-01",
		"horainicio": "1000",
		"horafin": "1500",
		"notas": "",
		"estado": "A",
		"comenta": "Va a ir rita",
		"tipo": "A",
		"tomas": "1",
		"volumen": "1",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "1/2 toma",
		"metodo": "",
		"suministrador": "",
		"modifica": "2023-07-31T13:46",
		"estadovisita": "OK"
	},
	{
		"idpeticion": "2",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2023-07-31T13:42",
		"fecha": "2023-08-01",
		"horainicio": "900",
		"horafin": "1400",
		"notas": "",
		"estado": "A",
		"comenta": "",
		"tipo": "I",
		"tomas": "0",
		"volumen": "0",
		"cantidad": "1",
		"material": "",
		"ensayos": "Acero",
		"capitulo": "2",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2023-07-31T13:43",
		"estadovisita": " "
	}
]
```
#### Caso 2: Filtrando -> Peticiones a las que se tiene acceso y que cumplen el filtro
**Solicitud:**
```json
{
	"obra": "13",
	"situa": "P",
	"visitaFin": "2025-05-31",
	"visitaIni": "2025-05-01"
}
```

**Respuesta:**
```json
[
	{
		"idpeticion": "9",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T11:36",
		"fecha": "2025-05-28",
		"horainicio": "1400",
		"horafin": "1500",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "A",
		"tomas": "1",
		"volumen": "1",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "Losa",
		"metodo": "Cami&oacute;n",
		"suministrador": "Hormisa Sur",
		"modifica": "2025-05-26T11:36",
		"estadovisita": ""
	},
	{
		"idpeticion": "16",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T13:56",
		"fecha": "2025-05-27",
		"horainicio": "1200",
		"horafin": "1400",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "I",
		"tomas": "0",
		"volumen": "0",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2025-05-26T13:56",
		"estadovisita": ""
	},
	{
		"idpeticion": "11",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T12:29",
		"fecha": "2025-05-27",
		"horainicio": "1200",
		"horafin": "1300",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "A",
		"tomas": "1",
		"volumen": "1",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "Losa",
		"metodo": "Cami&oacute;n",
		"suministrador": "Hormig&oacute;n del sur",
		"modifica": "2025-05-26T12:29",
		"estadovisita": ""
	},
	{
		"idpeticion": "15",
		"idobra": "13",
		"nombre": "Edificio Garraf  - Residencial de 6 plantas , Carrer Coronel San Feliu, 28, Cornell&aacute; de Llobregat",
		"alta": "2025-05-26T13:56",
		"fecha": "2025-05-27",
		"horainicio": "645",
		"horafin": "1000",
		"notas": "",
		"estado": "P",
		"comenta": "",
		"tipo": "R",
		"tomas": "0",
		"volumen": "0",
		"cantidad": "0",
		"material": "",
		"ensayos": "",
		"capitulo": "",
		"vertido": "",
		"metodo": "",
		"suministrador": "",
		"modifica": "2025-05-26T13:56",
		"estadovisita": ""
	}
]
```

## api/peticion/alta
**Parámetros de soliocitud**

```json
{
  "alta": "", //Una letra indica el tipo de trabajo a realizar: "A" amasada, "R" recogida de muestra, "I" ensayo in situ.
  "fecha": "",
	"horaFin": "",
	"horaInicio": "",
	"metodo": "",
	"movil": "",
	"notas": "",
	"obra": "",
  //Los siguientes datos son específicos para peticiones de tipo amasada:
	"suministrador": "",
	"tomas": "",
	"vertido": "",
	"volumen": ""
}
```

**Parámetros de respuesta**
- __id__: id de la petición grabada (para futuras consultas).
- __message__: Texto indicando el id de la petición grabada (ej. "Grabada petición número: 20").

#### Caso 1: Alta peticion de amasada

**Solicitud:**
```json
{
	"alta": "A",
	"fecha": "2025-05-30",
	"horaFin": "T15:00:00",
	"horaInicio": "T14:00:00",
	"metodo": "Camión",
	"movil": "666554433",
	"notas": "",
	"obra": "13",
	"suministrador": "Hormisa Sur",
	"tomas": "1",
	"vertido": "Losa",
	"volumen": "100"
}
```
**Respuesta:**
```json
{
	"id": 20,
	"message": "Grabada petición número: 20"
}
```

#### Caso 2: Alta peticion de recogida de muestras

**Solicitud:**
```json
{
	"alta": "R",
	"cantidad": "4",
	"fecha": "2025-05-31",
	"horaFin": "T12:00:00",
	"horaInicio": "T10:00:00",
	"material": "Acero",
	"movil": "666554433",
	"notas": "",
	"obra": "1"
}
```
**Respuesta:**
```json
{
	"id": 21,
	"message": "Grabada petición número: 21"
}
```

#### Caso 3: Alta peticion de ensayo in situ

**Solicitud:**
```json
{
	"alta": "I",
	"cantidad": "1",
	"capitulo": "1",
	"ensayos": "3",
	"fecha": "2025-06-05",
	"horaFin": "T14:00:00",
	"horaInicio": "T12:00:00",
	"movil": "666554433",
	"notas": "",
	"obra": "5"
}
```
**Respuesta:**
```json
{
	"id": 22,
	"message": "Grabada petición número: 22"
}
```


## api/peticion/\<id peticion\>/cancelar

**Parámetros de respuesta**
- __id__: id de la petición cancelada.
- __message__: Texto indicando el id de la petición cancelada (ej. "Grabada petición número: 20").

**Solicitud:**
Vacía

**Respuesta:**
```json
{
	"id": 21,
	"message": "La petición número 21 ha sido cancelada."
}
```
