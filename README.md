# Analizar datos de Facebook con Excel desde un archivo [JSON](https://developer.mozilla.org/es/docs/Learn/JavaScript/Objects/JSON)

### Fragmento de base de datos donde se almacenan las conversaciones de Facebook. Por privacidad suprimí el 99% de los mensajes.
###### Conversación propia (tipo de archivo JSON):
```json
{
  "participants": [
    {
      "name": "Saul LI Woon"
    },
    {
      "name": "St\u00c3\u00a9phano Soriano Urb\u00c3\u00a1n"
    }
  ],
  "messages": [
    {
      "sender_name": "Saul LI Woon",
      "timestamp_ms": 1553395762635,
      "content": "As\u00c3\u00ad es bro",
      "type": "Generic"
    },
    {
      "sender_name": "St\u00c3\u00a9phano Soriano Urb\u00c3\u00a1n",
      "timestamp_ms": 1553390894395,
      "content": "¿Trabajando?",
      "type": "Generic"
    },
    {
      "sender_name": "Saul LI Woon",
      "timestamp_ms": 1553216547218,
      "content": "Hola, muchas gracias bro ...",
      "type": "Generic"
    },
    {
      "sender_name": "St\u00c3\u00a9phano Soriano Urb\u00c3\u00a1n",
      "timestamp_ms": 1553216420364,
      "content": "Hola, espero est\u00c3\u00a9s bien",
      "type": "Generic"
    }
  ],
  "title": "Saul LI Woon",
  "is_still_participant": true,
  "thread_type": "Regular",
  "thread_path": "inbox/SaulLIWoon_1CFCpzMfww"
}
```
### Primer consulta 
###### Lenguaje M:
```js
let
    Origen = Json.Document(File.Contents("F:\OneDrive\Proyecto\FacebookProject\Query\LectureJSON\message_1.json")),
    messages = Origen[messages],
    #"Convertida en tabla" = Table.FromList(messages, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Se expandió Column1" = Table.ExpandRecordColumn(#"Convertida en tabla", "Column1", {"sender_name", "timestamp_ms", "content"}, {"Column1.sender_name", "Column1.timestamp_ms", "Column1.content"}),
    #"Columnas con nombre cambiado" = Table.RenameColumns(#"Se expandió Column1",{{"Column1.sender_name", "Emisor/Receptor"}, {"Column1.timestamp_ms", "Fecha"}, {"Column1.content", "Mensaje"}}),
    #"Personalizada agregada" = Table.AddColumn(#"Columnas con nombre cambiado", "Tiempo", each #datetime(1970, 1, 1, 0, 0, 0 ) + #duration(0, -6, 0, [Fecha]/1000)),
    #"Columna duplicada" = Table.DuplicateColumn(#"Personalizada agregada", "Tiempo", "Tiempo - Copia"),
    #"Hora insertada" = Table.AddColumn(#"Columna duplicada", "Hora", each DateTime.Time([#"Tiempo - Copia"]), type time),
    #"Fecha insertada" = Table.AddColumn(#"Hora insertada", "Fecha.1", each DateTime.Date([#"Tiempo - Copia"]), type date),
    #"Columnas reordenadas" = Table.ReorderColumns(#"Fecha insertada",{"Emisor/Receptor", "Fecha", "Mensaje", "Tiempo", "Tiempo - Copia", "Fecha.1", "Hora"}),
    #"Columnas quitadas" = Table.RemoveColumns(#"Columnas reordenadas",{"Fecha", "Tiempo", "Tiempo - Copia"}),
    #"Columnas con nombre cambiado1" = Table.RenameColumns(#"Columnas quitadas",{{"Fecha.1", "Fecha"}}),
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Columnas con nombre cambiado1",{{"Fecha", Int64.Type}})
in
    #"Tipo cambiado"
```
## Macro para guardar la base de datos de las conversaciones en formato CSV y para recodificar de "UTF-8" a "ISO-8859-1"
###### Lenguaje Visual Basic para Aplicaciones (VBA)
```vbnet
Sub Guardar_csv_y_recodificar()
'  Guardar_csv_y_recodificar Macro
'  Guarda una matriz de datos de un mensaje de Facebook Messenger en formato CSV y codificada a ISO-8859-1.
    ChDir "E:\"
    ActiveWorkbook.SaveAs Filename:= _
        "E:\messengerISO.csv" _
        , FileFormat:=xlCSV, CreateBackup:=False   
End Sub
```
### Segunda consulta 
###### Lenguaje M:`

```javascript
let
    Origen = Csv.Document(File.Contents("E:\messengerISO.csv"),[Delimiter=",", Columns=4, Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    #"Encabezados promovidos" = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
    #"Valor reemplazado" = Table.ReplaceValue(#"Encabezados promovidos","s�bado","sábado",Replacer.ReplaceText,{"Fecha"}),
    #"Valor reemplazado1" = Table.ReplaceValue(#"Valor reemplazado","mi�rcoles","miércoles",Replacer.ReplaceText,{"Fecha"})
in
    #"Valor reemplazado1"
```


### Gráfico 1. Tablero de información en Excel (en desarrollo)
###### Conversación de Facebook (propia) analizada:
<img src="https://github.com/StefanoSoriano/Analizar-datos-de-Facebook-con-Excel-en-lenguaje-M/blob/master/Facebook%20conversations.png?raw=true"/>

###### Fuente: Elaboración propia con datos de Facebook almacenados en un archivo .json

