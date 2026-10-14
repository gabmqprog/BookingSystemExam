# Felsökningsrapport – Skiftlägeskänslig namnsökning

## 1. Felet och hur jag återskapade det

Jag valde att undersöka felet i sökfunktionen, i metoden `Search()` i `BookingService.cs`. Problemet var att sökningen tog hänsyn till stora och små bokstäver, vilket gjorde att användaren inte alltid kunde hitta en registrerad bokning.

Jag testade detta manuellt genom att starta konsolprogrammet och välja alternativ 2, som är sökfunktionen. Jag sökte på `ava`, som är en del av kundnamnet `Avanoria AB`, men fick ingen träff trots att namnet fanns registrerat. Problemet var att det lilla `a` i söktexten inte matchade det stora `A` i kundnamnet.

Jag förväntade mig att sökfunktionen skulle hitta bokningen oavsett stora och små bokstäver, även när användaren bara skrev en del av kundnamnet. När sökningen inte hittade någon matchning returnerades en tom lista, vilket konsolprogrammet hanterar genom att visa meddelandet "No bookings were found."
## 2. Var felet fanns och varför det uppstod

Jag undersökte metoden `Search()` i `BookingService.cs` och hittade följande kod:

```csharp
booking.CustomerName.Contains(searchTerm)
```

`Contains()` används för att kontrollera om en text innehåller en viss söktext. Problemet var att den här jämförelsen var skiftlägeskänslig eftersom ingen annan jämförelse hade angetts.

Det innebar att programmet behandlade stora och små bokstäver som olika tecken. Om kunden hette `Avanoria AB` men användaren sökte på `ava`, matchade inte bokstäverna på grund av det stora `A` i kundnamnet.

## 3. Vad jag ändrade

Jag lade till `StringComparison.OrdinalIgnoreCase` i anropet till `Contains()`:

```csharp
booking.CustomerName.Contains(
    searchTerm,
    StringComparison.OrdinalIgnoreCase)
```

`Contains()` letar fortfarande efter söktexten i kundnamnet, men `StringComparison.OrdinalIgnoreCase` gör att sökningen inte skiljer mellan stora och små bokstäver.

Jag behövde inte ändra hur bokningarna visas i konsolen, eftersom problemet låg i själva jämförelsen i `Search()`.

## 4. Hur jag kontrollerade att det fungerade

Efter ändringen körde jag programmet igen och sökte på `ava`. Den här gången hittades bokningen för `Avanoria AB`.

Jag testade även att söka på `avanoria` med små bokstäver och fick rätt resultat. På så sätt kunde jag kontrollera att sökningen fungerade även när söktexten inte hade samma stora och små bokstäver som kundnamnet.

Jag körde också det automatiserade testet `Search_WhenSearchTermHasDifferentCasing_ReturnsBooking`, som blev grönt efter ändringen.

Om sökfunktionen skulle bli skiftlägeskänslig igen kan testet bli rött och visa att något inte fungerar som det ska. Testet kontrollerar nämligen att en bokning hittas även om söktexten innehåller andra stora och små bokstäver än kundnamnet.