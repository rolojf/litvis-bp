---
follows: pElias.md
---

## Análisis Producción Panel Solar Elías

```elm {l=hidden r}
type alias Panel =
    String


type alias Dia =
    Result (List Parser.DeadEnd) Calendar.Date


type alias Prod =
    Float


type alias Medicion =
    { panel : Panel
    , fecha : Dia
    , prod : Prod
    }


decodificaFecha =
    Decode.string
        |> Decode.column 1
        |> Decode.map Iso8601.toTime
        |> Decode.map
            (Result.map Calendar.fromPosix)


decoder =
    Decode.map3 Medicion
        (Decode.column 0 Decode.string)
        decodificaFecha
        (Decode.column 2 Decode.float)



--(Decode.column 1 Decode.string)
```

```elm {l r}
integrado : Result Decode.Error (List Medicion)
integrado =
    Decode.decodeCsv
        Decode.NoFieldNames
        decoder
        panelData
```

Muestra ^^^elm r=(List.map .fecha (Result.withDefault [] integrado))=^^^
