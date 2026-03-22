# 📚 Czyszczenie danych: transakcje finansowe

## Wstęp

Dane zostały wygenerowane maszynowo. Zawierają 10 000 wierszy na temat transakcji finansowych.

Do czyszczenia danych zostało wykorzystane Power Query oraz M-Language.

## Proces

1. Zmiana pierwszych wierszy na nagłówki danych oraz aby zaczynały się od wielkich liter.

```
#"Nagłówki o podwyższonym poziomie" = Table.PromoteHeaders(Źródło, [PromoteAllScalars=true]),

#"Nagłowki z wielkiej litery" = Table.TransformColumnNames(#"Zmieniono typ", Text.Proper),
```

2. Usunięcie duplikatów, sprawdzenie niepoprawnych danych.

3. Zmiana kolumny "Currency" na 2 odrębne kolumny, w których pierwsza ma kod waluty, a druga pełną nazwę danej waluty oraz usunięcie niepotrzebnych znaków.

```
#"Rozdzielenie waluty i nazwy" = Table.SplitColumn(
    Table.TransformColumns(#"Usunięto duplikaty",
                            {
                                "Currency",
                                each Text.Remove(_, {"(", ")", "'", """"})
                            }),
    "Currency",
    Splitter.SplitTextByDelimiter(", "), {"Currency_Code", "Currency_Name"}
    ),
```

4. W kolumnie "Receiver_Account" zmiana wartości null na "N/A"

```
#"Zamieniono wartość" = Table.ReplaceValue(#"Rozdzielenie waluty i nazwy","","N/A",Replacer.ReplaceValue,{"Receiver_Account"}),
```

5. W przypadku kolumn "Amount" i "Fees" zmiana typów danych. Aby to zrobić trzeba zamienić separatory na kropki.

```
#"Zmiana separatora" = Table.TransformColumns(
        #"Zamieniono wartość",
        {
            {"Amount", each Text.Replace(_, ".", ",")},
            {"Fees", each Text.Replace(_, ".", ",")}
        }
    ),
```

6. Dodanie kolumny niestandardowej jako całkowita suma transakcji ("Amount" + "Fees").

```
#"Zsumowanie kosztów transakcji" = Table.AddColumn(#"Zmieniono typ1", "Total Transaction Cost", each [Amount] + [Fees], Currency.Type)
```
