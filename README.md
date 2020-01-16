# Analizar datos de Facebook con Excel

Primer consulta (lenguaje M)

```javascript
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
## Conversación de Facebook (con uno de mis mejores amigos) analizada en Excel
<img src="https://github.com/StefanoSoriano/Analizar-datos-de-Facebook-con-Excel-en-lenguaje-M/blob/master/Facebook%20conversations.png?raw=true"/>

###### Fuente: Elaboración propia con datos de Facebook almacenados en un archivo .json

