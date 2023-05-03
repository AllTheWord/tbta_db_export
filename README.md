# The Bible Translator's Assistant (TBTA) DB Export

[TBTA](https://alltheword.org/our-downloads/) is a tool to assist Bible translation using a rules based system. An intermediate translation is necessary. This intermediary form takes additional information and attaches it to the words, phrases and clauses in the text. TBTA uses a format internally which is not very accessible to someone not familiar with the project. This repo and accompanying Livebook exports to JSON and XML which are more well known formats.

The naming scheme for the generated files is hopefully intuitive.

`book index number` _ `chapter` _ `verse` \_ `book name`

eg `00_001_001_Genesis.json` - Genesis 1:1

To use this, just clone this repo and use JSON or XML as suits your usecase. If that's all your after the rest of this readme is surplus to your needs.

## Livebook

If this is out of date and you want to export the data again, you can run the [Livebook](https://livebook.dev/run?url=https://github.com/AllTheWord/tbta_db_export/blob/main/tbta-db-export.livemd). [You can download Livebook here](https://livebook.dev/). Livebook is an Elixir tool but you shouldn't need to know Elixir. It's a standalone application if you are on Windows or Mac. If you're on Linux, that's an exercise for the reader. ;) Livebook is to Elixir what Jupyter is to Python so that may help give a frame of reference.

## mdbtools

This was initially performed on Linux. [`mdbtools`](https://github.com/mdbtools/mdbtools) is only on Linux and Mac as far as I know.

I have not provided a solution for Windows. You just need to be able to export `Bible.mdb` and put the results in `csv/Bible` and `Sample.mdb` and put the results in `csv/Grammar`. The latter actually just needs the `Features_Source` table.

After you have the CSV files in place, follow the rest of the Livebook that is not using `mdbtools`.

## Run it

After you have Livebook and `mdbtools` in place, you can just follow the instructions in the Livebook.

# Rabbit Holes Ahead

For documentation purposes and the excessively curious, here is the parsing strategy for TBTA. This is probably also useful background if you are trying to do some machine learning task.

Each constituent, phrase, subclause or clause is preceded by a string that represents the semantics embedded in the term about to follow. This string is split into individual characters and matched with its values from the target grammar database (`Sample.mdb` in our case). If you're going to do this yourself, it's important to pull the values from the database as sorted by their `ID`s.

Most of the elements are either wrapped in `~\wd ~\tg what you want ~\lu` or fall between them like `~\wd ~\tg before what you want ~\lu what you want ~\wd...` so that's the first step to understanding how to parse is you need to figure out how to capture these. For reference see the `Parse` module in the Livebook. For the matching the values from the semantic string to the DB values, it's the `Category` module. For the parsing of the semantic string into nested structures, etc., it's `Reshape`.

These examples are all taken from Ruth 3:17 unless otherwise noted. You can look at the appropriate file in [JSON](json/07_003_017_Ruth.json) or [XML](xml/07_003_017_Ruth.xml) for the parsed result.

## Noun - `SyntacticCategory` 1

`~\wd ~\tg N-1A1SDAnK3NN........~\lu Ruth~\wd ~\tg ~\lu`

`N-1A1SDAnK3NN........` is the semantic data

Let's refer to the positions by the index

0- `N` - Noun

1- `-` - Separator

##### These first three are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

4- `1` - Noun list index - useful for track of which nouns are referring to the same thing

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

5- `S` - Number - `S`: Singular, `D`: Dual, `T`: Trial, `Q`: Quadrial, `p`: Paucal, `P`: Plural

6- `D` - "Participant Tracking - `I`: First Mention, `D`: Routine, `i`: Integration, `E`: Exiting, `R`: Restaging, `O`: Offstage, `G`: Generic, `Q`: Interrogative, `F`: Frame Inferable

7- `A` - Polarity - `A`: Affirmative, `N`: Negative

8- `n` - "Proximity - `n`: Not Applicable, `N`: Near Speaker and Listener, `S`: Near Speaker, `L`: Near Listener, `R`: Remote within Sight, `r`: Remote out of Sight, `T`: Temporally Near, `t`: Temporally Remote, `C`: Contextually Near with Focus, `c`: Contextually Near

9- `K` - Future Expansion - `K`: Unspecified

10- `3` - Person - `1`: First, `2`: Second, `3`: Third, `A`: First Inclusive, `B`: First Exclusive, `F`: First as Third, `S`: Second as Third, `I`: First Inclusive as Third, `E`: First Exclusive as Third

11- `N` - Surface Realization - `N`: Noun, `A`: Always a Noun, `p`: PRO, `P`: Personal Pronoun

12- `N` - Participant Status - `N`: Not Applicable, `P`: Protagonist, `A`: Antagonist, `M`: Major Participant

13-22- Spares for future expansion

The value immediately following - `Ruth` is the constituent

##### Noun list index

At the end of the verse there is a list of nouns that this index refers to. Here is the raw data from Ruth 2:12:

`~\wd ~\tg c-IDpTenDONNNNNNNNN..............~\lu {~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A1SDAnK1NN........~\lu Boaz~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AEUINAN...........~\lu pray-hope~\wd ~\tg ~\lu )~\wd ~\tg c-PDpTenDONNNNNNNNN..............~\lu [~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A2SDAnK3NN........~\lu Yahweh~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AEUINAN...........~\lu do~\wd ~\tg ~\lu )~\wd ~\tg n-SPN.N........~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu good~\wd ~\tg ~\lu )~\wd ~\tg N-1B3PGAnK3NN........~\lu thing~\wd ~\tg ~\lu )~\wd ~\tg n-SBN.N........~\lu (~\wd ~\tg N-1A4SDAnK2NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg ~\lu ]~\wd ~\tg c-EDpTenDONNNNNNNNN..............~\lu [~\wd ~\tg P-1A.....~\lu because~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A4SDAnK2NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AeUINAN...........~\lu do~\wd ~\tg ~\lu )~\wd ~\tg n-SPN.N........~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu good~\wd ~\tg ~\lu )~\wd ~\tg N-1BAPGAnK3NN........~\lu thing~\wd ~\tg ~\lu )~\wd ~\tg ~\lu ]~\wd ~\tg .-~\lu .~\wd ~\tg ~\lu }~\wd ~\tg c-IDpTenDONNNNNNNNN..............~\lu {~\wd ~\tg C-1A.....~\lu and~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A1SDAnK1NN........~\lu Boaz~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AEUINAN...........~\lu pray-hope~\wd ~\tg ~\lu )~\wd ~\tg c-PDpTenDONNNNNNNNN..............~\lu [~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A2SDAnK3NN........~\lu Yahweh~\wd ~\tg c-tDpTenDONNNNNNNNNN.............~\lu [~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A2SDAcK3NN........~\lu Yahweh~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1BPUINAN...........~\lu be~\wd ~\tg ~\lu )~\wd ~\tg n-SSN.N........~\lu (~\wd ~\tg N-1A5SFAnK3NN........~\lu God~\wd ~\tg n-SNN.N........~\lu (~\wd ~\tg P-1A.....~\lu -Generic Genitive~\wd ~\tg N-1A6SDAnK3NN........~\lu Israel~\wd ~\tg ~\lu )~\wd ~\tg ~\lu )~\wd ~\tg ~\lu ]~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AEUINAN...........~\lu give~\wd ~\tg ~\lu )~\wd ~\tg n-SPN.N........~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu much-many~\wd ~\tg ~\lu )~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu good~\wd ~\tg ~\lu )~\wd ~\tg N-1ABPGAnK3NN........~\lu thing~\wd ~\tg ~\lu )~\wd ~\tg n-SdN.N........~\lu (~\wd ~\tg N-1A4SDAnK2NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg ~\lu ]~\wd ~\tg .-~\lu .~\wd ~\tg ~\lu }~\wd ~\tg c-IDpTenDONNNNNNNNN..............~\lu {~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A4SDAnK2NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AeUINAN...........~\lu come~\wd ~\tg ~\lu )~\wd ~\tg n-SdN.N........~\lu (~\wd ~\tg N-1A2SDAnK3NN........~\lu Yahweh~\wd ~\tg ~\lu )~\wd ~\tg c-EDpTenDONNNNNNNNN..............~\lu [~\wd ~\tg P-1A.....~\lu just-like~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu young~\wd ~\tg ~\lu )~\wd ~\tg N-1A7SIAnK3NN........~\lu bird~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1APUINAN...........~\lu come~\wd ~\tg ~\lu )~\wd ~\tg n-SdN.N........~\lu (~\wd ~\tg N-1A8SFAnK3NN........~\lu mother~\wd ~\tg n-SNN.N........~\lu (~\wd ~\tg P-1A.....~\lu -Kinship~\wd ~\tg N-1A7SDAnK3NN........~\lu bird~\wd ~\tg ~\lu )~\wd ~\tg ~\lu )~\wd ~\tg ~\lu ]~\wd ~\tg .-~\lu .~\wd ~\tg ~\lu }~\wd ~\tg c-IDpTenDONNNNNNNNN..............~\lu {~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A9PFAnK3NN........~\lu wing~\wd ~\tg n-SNN.N........~\lu (~\wd ~\tg P-1A.....~\lu -Body Part~\wd ~\tg N-1A8SDAnK3NN........~\lu mother~\wd ~\tg ~\lu )~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1APGINAN...........~\lu protect~\wd ~\tg ~\lu )~\wd ~\tg n-SPN.N........~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu young~\wd ~\tg ~\lu )~\wd ~\tg N-1A7SDAcK3NN........~\lu bird~\wd ~\tg ~\lu )~\wd ~\tg r-1A.....~\lu -QuoteEnd~\wd ~\tg .-~\lu .~\wd ~\tg ~\lu }~|ABoaz|AYahweh|Bthing|ARuth|AGod|AIsrael|Abird|Amother|Awing|Bthing|Athing|||||||||||||||||||||||||`

This list `~|ABoaz|AYahweh|Bthing|ARuth|AGod|AIsrael|Abird|Amother|Awing|Bthing|Athing|||||||||||||||||||||||||` is every noun in the verse in the order they appear. They are prefixed with their index of `A`, `B`, ...

The two `Bthing`s are referring to the same item as opposed to `Athing` which is a different item. This is useful for replacing with pronouns, etc. These values from the final string are not reflected in the JSON or XML apart from including their index.

## Verb - `SyntacticCategory` 2

`~\wd ~\tg V-1ArUINAN...........~\lu say~\wd ~\tg ~\lu`

`V-1ArUINAN...........` is the semantic data

Let's refer to the positions by the index

0- `V` - Verb

1- `-` - Separator

##### These first two are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

4- `r` - Time - `P`: Present, `D`: Immediate Past, `A`: Earlier Today, `a`: Yesterday, `b`: 2 Days Ago, `c`: 3 Days Ago, `d`: A Week Ago, `e`: A Month Ago, `f`: A Year Ago, `g`: During Speaker's Lifetime, `h`: Historic Past, `i`: Eternity Past, `q`: Unknown Past, `r`: Discourse, `E`: Immediate Future, `F`: Later Today, `j`: Tomorrow, `k`: 2 Days from Now, `l`: 3 Days from Now, `m`: A Week from Now, `n`: A Month from Now, `o`: A Year from Now, `p`: Unknown Future, `T`: Timeless

5- `U` - Aspect - `N`: Inceptive, `C`: Completive, `c`: Cessative, `o`: Continuative, `I`: Imperfective, `R`: Routinely, `H`: Habitual, `G`: Gnomic, `U`: Unmarked

6- `I` - Mood - `I`: Indicative, `a`: Definite Potential, `b`: Probable Potential, `c`: 'might' Potential, `j`: 'might not' Potential, `d`: Unlikely Potential, `e`: Impossible Potential, `f`: 'must' Obligation, `g`: 'should' Obligation, `h`: 'should not' Obligation, `i`: Forbidden Obligation, `l`: 'may' (permissive)

7- `N` - Reflexivity - `N`: Not Applicable, `R`: Reciprocal, `r`: Reflexive

8- `A` - Polarity - `A`: Affirmative, `N`: Negative, `E`: Emphatic Affirmative, `e`: Emphatic Negative

9- `N` - Adjective Degree - `N`: No Degree, `C`: Comparative, `S`: Superlative, `I`: Intensified, `E`: Extremely Intensified, `T`: 'too', `L`: 'less', `l`: 'least'

10- `.` - Target Tense & Form - `.`: Unspecified

11- `.` - Target Aspect - `N`: Inceptive, `C`: Completive, `c`: Cessative, `o`: Continuative, `I`: Imperfective, `R`: Routinely, `H`: Habitual, `G`: Gnomic, `U`: Unmarked, `.`: Unspecified

12- `.` - Target Mood - `I`: Indicative, `a`: Definite Potential, `b`: Probable Potential, `c`: 'might' Potential, `j`: 'might not' Potential, `d`: Unlikely Potential, `e`: Impossible Potential, `f`: 'must' Obligation, `g`: 'should' Obligation, `h`: 'should not' Obligation, `i`: Forbidden Obligation, `l`: 'may' (permissive), `.`: Unspecified

13-22- Spares for future expansion

The value immediately following - `say` is the constituent

## Adjective - `SyntacticCategory` 3

`~\wd ~\tg A-1AN.....~\lu 23~\wd ~\tg ~\lu`

`A-1AN.....` is the semantic data

Let's refer to the positions by the index

0- `A` - Adjective

1- `-` - Separator

##### These first two are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

4- `N` - Degree - `N`: No Degree, `C`: Comparative, `S`: Superlative, `I`: Intensified, `E`: Extremely Intensified, `T`: 'too', `L`: 'less', `l`: 'least', `q`: Equality, `i`: Intensified Comparative, `s`: Superlative of 2 items

5-9- Spares for future expansion

The value immediately following - `23` is the constituent.

## Adverb - `SyntacticCategory` 4

This example is from Ruth 1:14

`~\wd ~\tg a-1AN.....~\lu again~\wd ~\tg ~\lu`

`a-1AN.....` is the semantic data

Let's refer to the positions by the index

0- `a` - Adverb

1- `-` - Separator

##### These first two are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

4- `N` - Degree - `N`: No Degree, `C`: Comparative, `S`: Superlative, `V`: Intensified, `E`: Extremely Intensified, `T`: 'too', `L`: 'less', `l`: 'least'

5-9- Spares for future expansion

The value immediately following - `again` is the constituent

## Adposition - `SyntacticCategory` 5

`~\wd ~\tg P-1A.....~\lu -Kinship~\wd ~\tg`

`P-1A.....` is the semantic data

Let's refer to the positions by the index

0- `P` - Adposition

1- `-` - Separator

##### These first two are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

4-9- Spares for future expansion

The value immediately following - `-Kinship` is the constituent

## Conjunction - `SyntacticCategory` 6

`~\wd ~\tg C-1A.....~\lu and~\wd ~\tg`

`C-1A.....` is the semantic data

Let's refer to the positions by the index

0- `C` - Conjunction

1- `-` - Separator

##### These first two are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

4- `.` - Implicit - `.`: No, `Y`: Yes

5-9- Spares for future expansion

The value immediately following - `and` is the constituent

## Phrasal - `SyntacticCategory` 7

This example is from Exodus 3:17

`~\tg p-1A.....~\lu I am who I am~\wd ~\tg ~\lu`

`p-1A.....` is the semantic data

Let's refer to the positions by the index

0- `c` - Phrasal

1- `-` - Separator

##### These first two are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

4-9- Spares for future expansion

The value immediately following - `I am who I am` is the constituent

## Particle - `SyntacticCategory` 8

`~\wd ~\tg r-1A.....~\lu -QuoteBegin~\wd ~\tg`

`p-1A.....` is the semantic data

Let's refer to the positions by the index

0- `r` - Particle

1- `-` - Separator

##### These first two are not in the DB - they are hardcoded in TBTA

2- `1` - Semantic complexity level

3- `A` - Lexical sense

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

4-9- Spares for future expansion

The value immediately following - `-QuoteBegin` is the constituent

## Noun Phrase - `SyntacticCategory` 101

The phrases encapsulate other terms

`~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A1SDAnK3NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg`

`n-SAN.N........` is the semantic data

Let's refer to the positions by the index

0- `n` - Particle

1- `-` - Separator

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

2- `S` - Sequence - `S`: Not in a Sequence, `F`: First Coordinate, `L`: Last Coordinate, `C`: Coordinate

3- `A` - Semantic Role - `A`: Most Agent-like, `P`: Most Patient-like, `S`: State, `s`: Source, `d`: Destination, `I`: Instrument, `D`: Addressee, `B`: Beneficiary, `N`: Not Applicable

4- `N` - Implicit - `N`: Not Applicable, `G`: Explanation of Name, `A`: Optional Agent of Passive, `I`: Implicit Argument, `P`: Implicit Phrase, `M`: Explain Metonymy, `i`: Implicit but Necessary

5- `.` - Thing-Thing Relationship - `.`: Unspecified, `N`: Not Applicable

6- `N` - Relativized - `N`: No, `Y`: Yes

7-14- Spares for future expansion

`(` and `)` are the control characters that define the bounds of the phrase

## Verb Phrase - `SyntacticCategory` 102

The phrases encapsulate other terms

`~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1ArUINAN...........~\lu say~\wd ~\tg ~\lu )`

`v-S.....` is the semantic data

Let's refer to the positions by the index

0- `n` - Particle

1- `-` - Separator

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

2- `S` - Sequence - `S`: Not in a Sequence, `F`: First Coordinate, `L`: Last Coordinate, `C`: Coordinate

3- `.` - Implicit - `.`: No, `Y`: Yes, ", "Spare 2 - `.`: Unspecified

4-7 - Spares for future expansion

`(` and `)` are the control characters that define the bounds of the phrase

## Adjective Phrase - `SyntacticCategory` 103

The phrases encapsulate other terms

`~\tg j-SA.....~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu about~\wd ~\tg ~\lu )~\wd ~\tg A-1AN.....~\lu 23~\wd ~\tg ~\lu )`

`j-SA.....` is the semantic data

Let's refer to the positions by the index

0- `n` - Particle

1- `-` - Separator

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

2- `S` - Sequence - `S`: Not in a Sequence, `F`: First Coordinate, `L`: Last Coordinate, `C`: Coordinate

3- `A` - Usage - `A`: Attributive, `P`: Predicative, ", "Implicit - `.`: No, `Y`: Yes

4-7 - Spares for future expansion

`(` and `)` are the control characters that define the bounds of the phrase

## Adverb Phrase - `SyntacticCategory` 104

The phrases encapsulate other terms

`~\wd ~\tg d-S.....~\lu (~\wd ~\tg a-1AN.....~\lu again~\wd ~\tg ~\lu )`

`d-S.....` is the semantic data

Let's refer to the positions by the index

0- `d` - Particle

1- `-` - Separator

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

2- `S` - Sequence - `S`: Not in a Sequence, `F`: First Coordinate, `L`: Last Coordinate, `C`: Coordinate, ",

3- `.` - `Implicit - `.`: No, `Y`: Yes

4-7 - Spares for future expansion

`(` and `)` are the control characters that define the bounds of the phrase

## Clause - `SyntacticCategory` 105

The clauses encapsulate other terms

This is a clause, it is bounded by `{` and `}`:

`~\wd ~\tg c-IDp00NNNNNNNNNNNN..............~\lu {~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A1SDAnK3NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1ArUINAN...........~\lu say~\wd ~\tg ~\lu )~\wd ~\tg c-PDpAVnCYNNNNNNNNN..............~\lu [~\wd ~\tg r-1A.....~\lu -QuoteBegin~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A2SDAnK3NN........~\lu Boaz~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AAUINAN...........~\lu give~\wd ~\tg ~\lu )~\wd ~\tg n-SPN.N........~\lu (~\wd ~\tg N-1A3PGAnK3NN........~\lu grain/Bbarley~\wd ~\tg n-SNN.N........~\lu (~\wd ~\tg P-1A.....~\lu -Quantity~\wd ~\tg j-SA.....~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu about~\wd ~\tg ~\lu )~\wd ~\tg A-1AN.....~\lu 23~\wd ~\tg ~\lu )~\wd ~\tg N-1A4PGAnK3NN........~\lu kilogram~\wd ~\tg ~\lu )~\wd ~\tg ~\lu )~\wd ~\tg n-SdN.N........~\lu (~\wd ~\tg N-1A1SDAnK1NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg ~\lu ]~\wd ~\tg .-~\lu .~\wd ~\tg ~\lu }`

This is a subclause, embedded in the clause, it is bounded by `[` and `]`:

`~\tg c-PDpAVnCYNNNNNNNNN..............~\lu [~\wd ~\tg r-1A.....~\lu -QuoteBegin~\wd ~\tg n-SAN.N........~\lu (~\wd ~\tg N-1A2SDAnK3NN........~\lu Boaz~\wd ~\tg ~\lu )~\wd ~\tg v-S.....~\lu (~\wd ~\tg V-1AAUINAN...........~\lu give~\wd ~\tg ~\lu )~\wd ~\tg n-SPN.N........~\lu (~\wd ~\tg N-1A3PGAnK3NN........~\lu grain/Bbarley~\wd ~\tg n-SNN.N........~\lu (~\wd ~\tg P-1A.....~\lu -Quantity~\wd ~\tg j-SA.....~\lu (~\wd ~\tg j-SA.....~\lu (~\wd ~\tg A-1AN.....~\lu about~\wd ~\tg ~\lu )~\wd ~\tg A-1AN.....~\lu 23~\wd ~\tg ~\lu )~\wd ~\tg N-1A4PGAnK3NN........~\lu kilogram~\wd ~\tg ~\lu )~\wd ~\tg ~\lu )~\wd ~\tg n-SdN.N........~\lu (~\wd ~\tg N-1A1SDAnK1NN........~\lu Ruth~\wd ~\tg ~\lu )~\wd ~\tg ~\lu ]`

`c-IDp00NNNNNNNNNNNN..............` is the semantic data, clauses and and subclauses have the same semantic data structure

Let's refer to the positions by the index

0- `c` - Particle

1- `-` - Separator

##### These are pulled from the Grammar DB (`Sample.mdb`) - important to keep them in order

2- `I` - Type - `I`: Independent, `C`: Coordinate Independent, `T`: Restrictive Thing Modifier (Relative Clause), `t`: Descriptive Thing Modifier (Relative Clause), `E`: Event Modifier (Adverbial Clause), `A`: Agent (Subject Complement), `P`: Patient (Object Complement), `p`: Attributive Patient (Adjectival Object Complement), `Q`: Closing Quotation Frame

3- `D` - Illocutionary Force - `D`: Declarative, `I`: Imperative, `C`: Content Interrogative, `Y`: Yes-No Interrogative, `S`: Suggestive 'let's', `L`: Jussive, `i`: Imperative with emphasized Agent

4- `p` - Topic NP - `p`: Most Agent-like, `P`: Most Patient-like

5- `p` - Speaker - `0`: Not Applicable, `A`: Adult Daughter, `B`: Adult Son, `C`: Angel, `D`: Animal, `E`: Boy, `k`: Brother, `F`: Crowd, `g`: Daughter, `G`: Demon, `H`: Disciple, `I`: Employee, `J`: Employer, `K`: Father, `L`: Girl, `M`: God, `N`: Government Leader, `O`: Government Official, `j`: Group of Friends, `P`: Holy Spirit, `Q`: Husband, `R`: Jesus, `S`: King, `T`: Man, `U`: Military Leader, `V`: Mother, `W`: Prophet, `X`: Queen, `Y`: Religious Leader, `Z`: Satan, `f`: Servant, `l`: Sister, `a`: Slave, `b`: Slave Owner, `c`: Soldier, `h`: Son, `d`: Wife, `e`: Woman, `i`: Written Material to General Audience (letter,law,etc.)

6- `0` - Listener - `0`: Not Applicable, `A`: Adult Daughter, `B`: Adult Son, `C`: Angel, `D`: Animal, `E`: Boy, `k`: Brother, `F`: Crowd, `g`: Daughter, `G`: Demon, `H`: Disciple, `I`: Employee, `J`: Employer, `K`: Father, `L`: Girl, `M`: God, `N`: Government Leader, `O`: Government Official, `j`: Group of Friends, `P`: Holy Spirit, `Q`: Husband, `R`: Jesus, `S`: King, `T`: Man, `U`: Military Leader, `V`: Mother, `W`: Prophet, `X`: Queen, `Y`: Religious Leader, `Z`: Satan, `f`: Servant, `l`: Sister, `a`: Slave, `b`: Slave Owner, `c`: Soldier, `h`: Son, `d`: Wife, `e`: Woman, `m`: Nature (wind, waves)

7- `N` - Speaker's Attitude - `N`: Not Applicable, `n`: Neutral, `F`: Familiar, `E`: Endearing, `P`: Polite, `H`: Honorable, `f`: Friendly, `C`: Complimentary, `D`: Derogatory, `A`: Antagonistic-Hostile, `a`: Anger, `R`: Rebuke, `I`: Imploring

8- `N` - Speaker's Age - `N`: Not Applicable, `A`: Child (0-17), `B`: Young Adult (18-24), `C`: Adult (25-49), `D`: Elder (50+)

9- `N` - Speaker-Listener Age - `N`: Not Applicable, `O`: Older - Different Generation, `o`: Older - Same Generation, `S`: Essentially the Same Age, `Y`: Younger - Different Generation, `y`: Younger - Same Generation

10- `N` - Speech Style - `N`: Not Applicable

11- `N` - Discourse Genre - `N`: Climactic Narrative Story, `n`: Episodic Narrative Story, `r`: Narrative Prophecy, `E`: Expository, `H`: Behavioral Hortatory, `B`: Behavioral Eulogy, `P`: Procedural, `R`: Persuasive, `e`: Expressive, `D`: Descriptive, `p`: Epistolary, `d`: Dramatic Narrative, `G`: Dialog, `g`: Genealogy

12- `N` - Notional Structure Schema - `N`: Not Applicable, `E`: Narrative-Exposition, `I`: Narrative-Inciting Incident, `d`: Narrative-Developing Conflict, `C`: Narrative-Climax, `D`: Narrative-Denouement, `F`: Narrative-Final Suspense, `c`: Narrative-Conclusion, `A`: Hortatory-Authority Establishment, `P`: Hortatory-Problem or Situation, `i`: Hortatory-Issuing of Commands, `M`: Hortatory-Motivation, `p`: Procedural-Problem or Need, `a`: Procedural-Preparatory Procedures, `b`: Procedural-Main Procedures, `e`: Procedural-Concluding Procedures, `f`: Persuasive-Problem or Question, `g`: Persuasive-Proposed Solution or Answer, `h`: Persuasive-Supporting Argumentation, `j`: Persuasive-Appeal, `k`: Expository-Problem or Situation, `m`: Expository-Solution or Answer, `n`: Expository-Supporting Argumentation, `o`: Expository-Evaluation of Solutions

13- `N` - Salience Band - `N`: Not Applicable, `1`: Pivotal Storyline, `P`: Primary Storyline, `2`: Secondary Storyline, `S`: Script Predictable Actions, `B`: Backgrounded Actions, `F`: Flashback, `s`: Setting, `I`: Irrealis, `E`: Author Intrusion, `C`: Cohesive Material

14- `N` - Sequence - `N`: Not in a Sequence, `F`: First Coordinate, `L`: Last Coordinate, `C`: Coordinate

15- `N` - Location - `N`: Not Applicable, `F`: First in Book, `L`: Last in Book, `f`: First in Paragraph, `l`: Last in Paragraph, `T`: Discourse Title, `P`: Parenthetical Comment, `G`: General Footnote, `o`: Footnote Explaining a Hebrew Name, `M`: Footnote Explaining a Biblical Unit of Measurement, `C`: Cross Reference Footnote, `S`: Study Footnote, `D`: Devotional Footnote, `Q`: Questionable Text, `r`: Prepose Short Sentence, `p`: Postpose Short Sentence

16- `N` - Implicit Information - `N`: Not Applicable, `C`: Implicit Cultural Information, `S`: Implicit Situational Information, `H`: Implicit Historical Information, `B`: Implicit Background Information, `s`: Implicit Subactions, `A`: Implicit Argument

17- `N` - Alternative Analysis - `N`: Not Applicable, `P`: Primary Analysis, `A`: First Alternative Analysis, `B`: Second Alternative Analysis, `C`: Third Alternative Analysis, `D`: Fourth Alternative Analysis, `E`: Fifth Alternative Analysis, `L`: Literal Alternate, `d`: Dynamic Alternate, `b`: Biblical Units, `c`: Contemporary Units

18- `N` - Vocabulary Alternate - `N`: Not Applicable, `V`: Single Sentence - Complex Vocabulary Alternate, `F`: First Sentence - Complex Vocabulary Alternate, `L`: Last Sentence - Complex Vocabulary Alternate, `C`: Coordinate Sentence - Complex Vocabulary Alternate, `s`: Single Sentence - Simple Vocabulary Alternate, `f`: First Sentence - Simple Vocabulary Alternate, `l`: Last Sentence - Simple Vocabulary Alternate, `c`: Coordinate Sentence - Simple Vocabulary Alternate

19- `N` - Rhetorical Question - `N`: Not Applicable, `Q`: Content Question, `Y`: Yes-No Question Expects 'Yes', `n`: Yes-No Question Expects 'No', `S`: Equivalent Statement

20-33 - Spares for future expansion

## Paragraph - `SyntacticCategory` 110

A paragraph is just a marker to show the beginning of a paragraph; it does not mark the end

A paragraph marker immediately precedes a clause

This example is from 2 Samuel 9:1

`~\wd ~\tg R-~\lu |~\wd ~\tg c-IDp00NNNNNNNNNNNNN.............~\lu {`

`R-` is the semantic data - it's empty

## Episode - `SyntacticCategory` 120

There are no examples of the episode marker in the derived data but if there were it would look much like the paragraph apart from using a different letter

It would look like this

`~\wd ~\tg E-~\lu |~\wd ~\tg c-IDp00NNNNNNNNNNNNN.............~\lu {`
