---
elm:
  dependencies:
    elm/html: latest
    elm/json: latest
    elm/svg: latest
    elm/url: latest
    elm/virtual-dom: latest
    ianmackenzie/elm-1d-parameter: latest
    ianmackenzie/elm-geometry: latest
    ianmackenzie/elm-units: latest
  source-directories:
    - ./vendor
    - ./elm-svg-string/src
---

## Proyecto para comprobar si SVG en este sistema

1. La idea crear geometrías con los módulos de IanMackenzie y desplegarlos como
   SVG. Internamente, los convertimos de Svg a String usando el módulo
   modificado de `PaackEng/elm-svg-string` que está en el subdirectorio del
   mismo nombre.

```elm {l=hidden}
--import Html.String as Html exposing (Html)
--import Html.String.Attributes as Attr


import Angle exposing (Angle)
import Arc2d exposing (Arc2d)
import Axis2d exposing (Axis2d)
import Circle2d exposing (Circle2d)
import CubicSpline2d exposing (CubicSpline2d)
import Dict
import Direction2d exposing (Direction2d)
import Ellipse2d exposing (Ellipse2d)
import EllipticalArc2d exposing (EllipticalArc2d)
import Frame2d exposing (Frame2d)
import LineSegment2d exposing (LineSegment2d)
import Parameter1d
import Pixels exposing (Pixels)
import Point2d exposing (Point2d)
import Polygon2d exposing (Polygon2d)
import Polyline2d exposing (Polyline2d)
import QuadraticSpline2d exposing (QuadraticSpline2d)
import Quantity exposing (Quantity)
import Rectangle2d exposing (Rectangle2d)
import SSvg as SvgS
import Svg.String as SvgS
import Svg.String.Attributes as Attributes
import Triangle2d exposing (Triangle2d)
import Vector2d exposing (Vector2d)
```

### Definir las funciones que grafican

En sección triple backtick, practicamente usando solo con lo del módulo de
ianmackenzie definimos geometrias.

```elm {l=hidden}
point2d : List (SvgS.Attribute msg) -> Point2d Pixels coordinates -> SvgS.Svg msg
point2d attributes point =
    point2dWith { radius = Pixels.pixels 3 } attributes point


point2dWith : { radius : Quantity Float Pixels } -> List (SvgS.Attribute msg) -> Point2d Pixels coordinates -> SvgS.Svg msg
point2dWith { radius } attributes point =
    SvgS.circle2d attributes (Circle2d.withRadius radius point)


circle : SvgS.Svg msg
circle =
    SvgS.circle2d
        [ Attributes.fill "orange"
        , Attributes.stroke "blue"
        , Attributes.strokeWidth "2"
        ]
        (Circle2d.withRadius (Pixels.pixels 10) (Point2d.pixels 150 150))


ellipse : SvgS.Svg msg
ellipse =
    SvgS.ellipse2d
        [ Attributes.fill "orange"
        , Attributes.stroke "blue"
        , Attributes.strokeWidth "2"
        ]
        (Ellipse2d.with
            { centerPoint = Point2d.pixels 150 150
            , xDirection = Direction2d.degrees -30
            , xRadius = Pixels.pixels 60
            , yRadius = Pixels.pixels 30
            }
        )


lineSegment : SvgS.Svg msg
lineSegment =
    SvgS.lineSegment2d
        [ Attributes.stroke "blue"
        , Attributes.strokeWidth "5"
        ]
        (LineSegment2d.from
            (Point2d.pixels 100 100)
            (Point2d.pixels 200 200)
        )


triangle : SvgS.Svg msg
triangle =
    SvgS.triangle2d
        [ Attributes.stroke "blue"
        , Attributes.strokeWidth "10"
        , Attributes.strokeLinejoin "round"
        , Attributes.fill "orange"
        ]
        (Triangle2d.fromVertices
            ( Point2d.pixels 100 100
            , Point2d.pixels 200 100
            , Point2d.pixels 100 200
            )
        )


polyline : SvgS.Svg msg
polyline =
    SvgS.polyline2d
        [ Attributes.stroke "blue"
        , Attributes.fill "none"
        , Attributes.strokeWidth "5"
        , Attributes.strokeLinecap "round"
        , Attributes.strokeLinejoin "round"
        ]
        (Polyline2d.fromVertices
            [ Point2d.pixels 100 100
            , Point2d.pixels 120 200
            , Point2d.pixels 140 100
            , Point2d.pixels 160 200
            , Point2d.pixels 180 100
            , Point2d.pixels 200 200
            ]
        )


polygon : SvgS.Svg msg
polygon =
    SvgS.polygon2d
        [ Attributes.stroke "blue"
        , Attributes.fill "orange"
        , Attributes.strokeWidth "3"
        ]
        (Polygon2d.withHoles
            [ [ Point2d.pixels 150 185
              , Point2d.pixels 165 160
              , Point2d.pixels 135 160
              ]
            ]
            [ Point2d.pixels 100 200
            , Point2d.pixels 120 150
            , Point2d.pixels 180 150
            , Point2d.pixels 200 200
            ]
        )


rectangle : SvgS.Svg msg
rectangle =
    let
        axes =
            Frame2d.atPoint (Point2d.pixels 150 150)
                |> Frame2d.rotateBy (Angle.degrees 20)
    in
    SvgS.rectangle2d
        [ Attributes.stroke "blue"
        , Attributes.fill "orange"
        , Attributes.strokeWidth "4"
        , Attributes.rx "15"
        , Attributes.ry "15"
        ]
        (Rectangle2d.centeredOn axes ( Pixels.pixels 120, Pixels.pixels 80 ))


arc : SvgS.Svg msg
arc =
    SvgS.arc2d
        [ Attributes.stroke "blue"
        , Attributes.strokeWidth "5"
        , Attributes.fill "none"
        , Attributes.strokeLinecap "round"
        ]
        (Point2d.pixels 150 75
            |> Arc2d.sweptAround (Point2d.pixels 100 100)
                (Angle.degrees 135)
        )


ellipticalArc : SvgS.Svg msg
ellipticalArc =
    SvgS.ellipticalArc2d
        [ Attributes.stroke "blue"
        , Attributes.fill "none"
        , Attributes.strokeWidth "5"
        , Attributes.strokeLinecap "round"
        ]
        (EllipticalArc2d.with
            { centerPoint = Point2d.pixels 100 10
            , xDirection = Direction2d.x
            , xRadius = Pixels.pixels 50
            , yRadius = Pixels.pixels 100
            , startAngle = Angle.degrees 0
            , sweptAngle = Angle.degrees 180
            }
        )


quadraticSpline : SvgS.Svg msg
quadraticSpline =
    let
        firstControlPoint =
            Point2d.pixels 50 50

        secondControlPoint =
            Point2d.pixels 100 150

        thirdControlPoint =
            Point2d.pixels 150 100

        spline =
            QuadraticSpline2d.fromControlPoints
                firstControlPoint
                secondControlPoint
                thirdControlPoint

        controlPoints =
            [ firstControlPoint, secondControlPoint, thirdControlPoint ]

        drawPoint point =
            SvgS.circle2d [] <|
                Circle2d.withRadius (Pixels.pixels 3) point
    in
    SvgS.g [ Attributes.stroke "blue" ]
        [ SvgS.quadraticSpline2d
            [ Attributes.strokeWidth "3"
            , Attributes.strokeLinecap "round"
            , Attributes.fill "none"
            ]
            spline
        , SvgS.polyline2d
            [ Attributes.strokeWidth "1"
            , Attributes.fill "none"
            , Attributes.strokeDasharray "3 3"
            ]
            (Polyline2d.fromVertices controlPoints)
        , SvgS.g [ Attributes.fill "white" ]
            (List.map drawPoint controlPoints)
        ]


cubicSpline : SvgS.Svg msg
cubicSpline =
    let
        firstControlPoint =
            Point2d.pixels 50 50

        secondControlPoint =
            Point2d.pixels 100 150

        thirdControlPoint =
            Point2d.pixels 150 25

        fourthControlPoint =
            Point2d.pixels 200 125

        spline =
            CubicSpline2d.fromControlPoints
                firstControlPoint
                secondControlPoint
                thirdControlPoint
                fourthControlPoint

        controlPoints =
            [ firstControlPoint
            , secondControlPoint
            , thirdControlPoint
            , fourthControlPoint
            ]

        drawPoint point =
            SvgS.circle2d [] <|
                Circle2d.withRadius (Pixels.pixels 3) point
    in
    SvgS.g [ Attributes.stroke "blue" ]
        [ SvgS.cubicSpline2d
            [ Attributes.strokeWidth "3"
            , Attributes.strokeLinecap "round"
            , Attributes.fill "none"
            ]
            spline
        , SvgS.polyline2d
            [ Attributes.strokeWidth "1"
            , Attributes.fill "none"
            , Attributes.strokeDasharray "3 3"
            ]
            (Polyline2d.fromVertices controlPoints)
        , SvgS.g [ Attributes.fill "white" ]
            (List.map drawPoint controlPoints)
        ]


scaled : SvgS.Svg msg
scaled =
    let
        scales =
            [ 1.0, 1.5, 2.25 ]

        referencePoint =
            Point2d.pixels 100 100

        scaledCircle : Float -> SvgS.Svg msg
        scaledCircle scale =
            SvgS.scaleAbout referencePoint scale circle
    in
    SvgS.g []
        (point2d [ Attributes.fill "black" ] referencePoint
            :: List.map scaledCircle scales
        )


rotated : SvgS.Svg msg
rotated =
    let
        angles =
            Parameter1d.steps 9 <|
                Quantity.interpolateFrom
                    (Angle.degrees 0)
                    (Angle.degrees 270)

        referencePoint =
            Point2d.pixels 200 150

        rotatedCircle : Angle -> SvgS.Svg msg
        rotatedCircle angle =
            SvgS.rotateAround referencePoint angle circle
    in
    SvgS.g []
        (point2d [ Attributes.fill "black" ] referencePoint
            :: List.map rotatedCircle angles
        )


translated : SvgS.Svg msg
translated =
    SvgS.g []
        [ polyline
        , SvgS.translateBy (Vector2d.pixels 0 40) polyline
        , SvgS.translateBy (Vector2d.pixels 5 -60) polyline
        ]


mirrored : SvgS.Svg msg
mirrored =
    let
        horizontalAxis =
            Axis2d.through (Point2d.pixels 0 220) Direction2d.x

        horizontalAxisSegment =
            LineSegment2d.along horizontalAxis
                (Pixels.pixels 50)
                (Pixels.pixels 250)

        angledAxis =
            Axis2d.through (Point2d.pixels 0 150)
                (Direction2d.degrees -10)

        angledAxisSegment =
            LineSegment2d.along angledAxis
                (Pixels.pixels 50)
                (Pixels.pixels 250)
    in
    SvgS.g []
        [ polygon
        , SvgS.mirrorAcross horizontalAxis polygon
        , SvgS.mirrorAcross angledAxis polygon
        , SvgS.g
            [ Attributes.strokeWidth "0.5"
            , Attributes.stroke "black"
            , Attributes.strokeDasharray "3 3"
            ]
            [ SvgS.lineSegment2d [] horizontalAxisSegment
            , SvgS.lineSegment2d [] angledAxisSegment
            ]
        ]


placed : SvgS.Svg msg
placed =
    let
        stamp =
            SvgS.polygon2d
                [ Attributes.fill "orange"
                , Attributes.stroke "blue"
                , Attributes.strokeWidth "2"
                ]
                (Polygon2d.singleLoop
                    [ Point2d.origin
                    , Point2d.pixels 40 0
                    , Point2d.pixels 50 25
                    , Point2d.pixels 10 25
                    ]
                )

        frames =
            [ Frame2d.atPoint (Point2d.pixels 25 25)
            , Frame2d.atPoint (Point2d.pixels 100 25)
            , Frame2d.atPoint (Point2d.pixels 175 25)
                |> Frame2d.rotateBy (Angle.degrees 20)
            , Frame2d.atPoint (Point2d.pixels 25 150)
            , Frame2d.atPoint (Point2d.pixels 100 100)
                |> Frame2d.rotateBy (Angle.degrees 20)
            , Frame2d.atPoint (Point2d.pixels 150 150)
                |> Frame2d.rotateBy (Angle.degrees -30)
            ]
    in
    SvgS.g [] (List.map (\frame -> SvgS.placeIn frame stamp) frames)
```

### Luego definimos las imagenes y como se muestran

Usando funciones de geometrías y convirtiendo a `SvgS.Html msg`.

```elm {l=hidden}
example : ( Float, Float ) -> ( Float, Float ) -> SvgS.Svg msg -> SvgS.Html msg
example ( minX, minY ) ( maxX, maxY ) someSvg =
    let
        topLeftFrame =
            Frame2d.atPoint (Point2d.pixels minX maxY)
                |> Frame2d.reverseY

        width =
            maxX - minX

        height =
            maxY - minY
    in
    SvgS.svg
        [ Attributes.width (String.fromFloat width)
        , Attributes.height (String.fromFloat height)

        --, Attr.style "display" "block"
        ]
        [ someSvg |> SvgS.relativeTo topLeftFrame ]


examples : List ( String, SvgS.Html msg )
examples =
    [ ( "circle", example ( 130, 130 ) ( 170, 170 ) circle )
    , ( "ellipse", example ( 80, 100 ) ( 220, 200 ) ellipse )
    , ( "lineSegment", example ( 90, 90 ) ( 210, 210 ) lineSegment )
    , ( "triangle", example ( 90, 90 ) ( 210, 210 ) triangle )
    , ( "polyline", example ( 90, 90 ) ( 210, 210 ) polyline )
    , ( "polygon", example ( 90, 140 ) ( 210, 210 ) polygon )
    , ( "rectangle", example ( 50, 75 ) ( 250, 225 ) rectangle )
    , ( "arc", example ( 70, 60 ) ( 170, 170 ) arc )
    , ( "ellipticalArc", example ( 40, 0 ) ( 160, 120 ) ellipticalArc )
    , ( "quadraticSpline", example ( 35, 35 ) ( 165, 165 ) quadraticSpline )
    , ( "cubicSpline", example ( 30, 5 ) ( 220, 170 ) cubicSpline )
    , ( "scaled", example ( 90, 90 ) ( 250, 250 ) scaled )
    , ( "rotated", example ( 130, 80 ) ( 270, 220 ) rotated )
    , ( "translated", example ( 87, 25 ) ( 215, 255 ) translated )
    , ( "mirrored", example ( 35, 20 ) ( 265, 300 ) mirrored )
    , ( "placed", example ( 10, 10 ) ( 235, 190 ) placed )
    ]


mostrar =
    List.map
        (Tuple.second >> SvgS.toString 0)
        examples
```

## Finalmente, todo pre-definido mostramos

Utilizando para deplegar de LitVis triple hat notation m=List delString.

Ahora pues: Con texto previsamente definido.

Ver ^^^elm m=mostrar^^^

#### FIN
