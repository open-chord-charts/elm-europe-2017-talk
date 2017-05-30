## Open Chords Charts

![](assets/logo.png)

I'm Christophe Benz

developer and jazz pianist

contact@cbenz.org

---

Jam session!

![](assets/cbenz-piano.jpg)

Note:
- I'm also a pianist who loves Jazz and improvised music
- Happy and grateful to be here
- I will talk about music and Elm for 20 minutes

---

## Chords charts?

- `song = melody + chords + tonality`
- applies to rock, blues, jazz, folk
- transposing is changing tonality

Note:
- when I want to play a song, besides the melody, I need chords
  - rock, blues, jazz, folk – same principle!
- I'd like to have a repertoire of songs
  - with transpose feature: change the tonality dynamically
- Can't to that with images or plain text
- I didn't find any free software doing this, and not in Elm ;-)

+++

## Example: Jazz song

![All of me](assets/all-of-me.png)

It's an image :-(

Note:
- it's a convenient way to represent a song
- keeping only chords
- presented as a table
- does not include the melody

+++

![Guitar chords](assets/guitar-chords.jpg)

We just keep the "letters"

+++

## Focus on the first chord

<span style="font-size: 3em;">C</span>

Means "C Major"

+++

## Another one

<span style="font-size: 3em;">Fm</span>

Means "F minor"

---

## What is a chord?

- a "root" music note
  - A, B, C, D, E, F, G
- a quality
  - Major, minor, etc.

+++

## Music notes in Elm

```elm
type Note
    = A | Af | As
    | B | Bf | Bs
    | C | Cf | Cs
    ...
    | F | Ff | Fs
    | G | Gf | Gs

type alias OctaveIndex = Int

toOctaveIndex : Note -> OctaveIndex

toOctaveIndex Es == toOctaveIndex Ff -- enharmonics
```

Note:
- first using Int but doesn't allow enharmonics
- then using ADT
- refactoring was secured by the Elm compiler

+++

## Chords in Elm

```elm
type Chord
    = Chord Note Quality

type Quality
    = Major
    | Minor
    | Augmented
    | MajorSixth
    | MinorSixth
    | Seventh
    ...
```

Note:
- qualities are just labels corresponding to commonly used chords

+++

## Chords Charts in Elm

![All of me](assets/all-of-me.png)

+++

## Chords Charts in Elm

```elm
type alias Chart =
    { title : String
    , tonality : Note
    , parts : List Part
    }

type Part
    = Part String (List Bar)
    | PartRepeat String
```

+++

## Bars examples

![](https://raw.githubusercontent.com/open-chords-charts/chart-dsl/master/grammar-images/chords/major-triad.png)
![](https://raw.githubusercontent.com/open-chords-charts/chart-dsl/master/grammar-images/bar-repeat-1-chord.png)
![](https://raw.githubusercontent.com/open-chords-charts/chart-dsl/master/grammar-images/bar-2-chords.png)
![](https://raw.githubusercontent.com/open-chords-charts/chart-dsl/master/grammar-images/bar-3-chords.png)
![](https://raw.githubusercontent.com/open-chords-charts/chart-dsl/master/grammar-images/bar-4-chords.png)

Note:
- I set a limit of 4 chords by bar

+++

## Bars in Elm

```elm
type Bar
    = Bar (List Chord)
    | BarRepeat
```

+++

## Transpose a chords chart

```elm
transpose : Note -> Chart -> Chart
transpose key chart =
    let
        interval =
            Note.interval chart.key key

        newParts =
            chart.parts |> List.map (transposePart interval)
    in
        { chart
            | key = key
            , parts = newParts
        }
```

Note:
- transposing a chords chart is basically transposing its parts, and setting the new key

+++

## Map a function on part bars

```elm
-- reminder
type Part
    = Part String (List Bar)
    | PartRepeat String

mapPartBars : (List Bar -> List Bar) -> Part -> Part
mapPartBars f part =
    case part of
        Part partName bars ->
            Part partName (f bars)

        PartRepeat _ ->
            part
```

Note:
- I assume you already see what's the `List.map` function is
- Here we abstract a pattern, applying a function over the bars of a part
  - only if it's a normal part, not a repeated part

+++

## Transpose a part

```elm
transposePart : Interval -> Part -> Part
transposePart interval part =
    part |> mapPartBars (List.map (transposeBar interval))

transposeBar : Interval -> Bar -> Bar
transposeBar interval bar =
    bar |> mapBarChords (List.map (Chord.transpose interval))
```

+++

## Transpose a chord

```elm
-- Music.Chord
transpose : Interval -> Chord -> Chord
transpose interval (Chord note quality) =
    Chord (Note.transpose interval note) quality

-- Music.Note
transpose : Interval -> Note -> Note
transpose interval note =
    let
        octaveIndex =
            toOctaveIndex note
    in
        fromOctaveIndex (octaveIndex + interval)
```

---

## Chart viewer / editor

![](assets/all-of-me-c.png)

+++

```elm
allOfMe : Chart
allOfMe =
    let
        partA = Part "A"
            [ Bar [ Chord C Major ]
            , BarRepeat
            ...
            ]
        partB = ...
        partC = ...
    in
        { title = "All of me", key = C
        , parts = [ partA, partB, PartRepeat "A", partC ]
        }
```

+++

## Chart viewer / editor

```elm
type alias Model =
    { chart : Chart
    , viewedKey : Note
    }

view model =
    let
        viewedChart =
            model.chart
                |> Music.Chart.transpose model.viewedKey
    in
        ...
```

Note:
- viewed key is different from chart key

+++

## Chart viewer / editor

![](assets/all-of-me-g.png)

+++

## Chart viewer / editor

# Demo

Note:
- show
  - click on edit
  - select a bar
  - change a chord
  - set a bar repeat
  - click on save
  - change the key

+++

## Chart viewer / editor

```elm
type Msg
    = Edit
    | Save
    | SelectBar BarReference
    | SetChord BarReference ChordIndex Chord
    | SetBarRepeat BarReference Bool
    | SetViewKey Note

type alias BarReference =
    { partIndex : PartIndex
    , barIndex : BarIndex
    }
```

---

## A Domain Specific Language

```
title: All of me
key: C

= A
C - E7 - A7 - Dm -

= B
E7 - Am - D7 - G7 -

= A

= C
F Fm C A7 Dø G7 C -
```

Note:
- started as an experiment
- human-friendly serialization
- people can share a chords chart by email in plain text

+++

### elm-tools/parser

```elm
chart : Parser Chart
chart =
    inContext "chart" <|
        succeed Chart
            |. spacesAndNewlines
            |. symbol "title:"
            |. spaces
            |= keepUntilEndOfLine
            |. newLine
            |. symbol "key:"
            |. spaces
            |= note
            |. spacesAndNewlines
            |= repeat oneOrMore (part |. spacesAndNewlines)
            |. end
```

Note:
- almost same complexity than JSON decoders
- JSON encoders still needed for a web API in order to be easily consumable by other programming languages

---

# Conclusion

- Elm is awesome to model a domain like music
- Refactoring is a real pleasure
- Just the beginning!
- Questions?

Note:
- TODO
  - web API
  - public Single-Page Application website with authentication
    - thanks to https://github.com/rtfeldman/elm-spa-example
  - offline mode
    - by downloading a snapshot of the chords charts in local storage?
    - using Progressive Web Apps?
