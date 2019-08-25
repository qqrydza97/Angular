## Components

#### Właściwości komponentu:

- Serwisy mogą być wstrzykiwane bezpośredni do komponentu
- Mogą do nich być przekazywane dane z komponentów nadrzędnych za pomocą wejść
komponentu 
- Hermetyzują style
- Animacja Angulara
- Wyjścia są właściwościami powiązanymi ze zdarzeniami. Są wykorzystane przez
komponenty nadrzędne do wykrywania zmian w tym komponencie
- Zaczep cyklu życia (lifecycle hooks)

#### Sposoby odwołania się do komponentów potomnych (zagnieżdżanie komponentów):

1. Deklarowanie innego komponentu w templecie innego -> @ViewChild
2. Zawartość, która ma zostać wstawiona do jego komponentu -> @ContentChild

#### Rodzaje komponentów:

* Komponent App
* Komponent Route
* Komponent Display – komponent bezstanowy, który odzwierciedla przekazywane do
  niego wartości. Dane zawsze są przekazywane przez bindowanie, zamiast
  dynamicznie pobierać dane. Te komponenty budują swoje API. Akceptują bindowania
  wejściowe i emitują zdarzenia dla akcji, które odsyłają w celu zmiany danych.
* Komponent Data – przechowuje dane lub logikę biznesową. W tych komponentach
  często wykorzystywane są zaczepy cyklu życia, by uruchomić ładowanie i utratę
  danych. Należy próbować ograniczyć ilość komponentów, które mają do czynienia z
  danymi.

#### Punkt wstawienia:

`ng-content` – Obszar akceptujący wszelkiego rodzaju znaczniki. W tym przypadku templete komponentu app-container ma zagnieżdzony `ng-content`. Do tego kontentu jest wstawiny zagnieżdzony komponent angulara `app-inner`. 

```html
<app-container>
    <app-inner></app-inner>
</app-container>
```
## Zawansowane aspekty komponentów

#### Sposoby renderowania reagując na zmiany
Po uruchomieniu mechanizmu wykrywania zmian rozpocznie działanie od wierzchołka drzewa komponentów i będzie sorawdzał każdy węzeł, aby zobaczyć, czy model komponentu nie uległ zmianie i nie wymaga renderowania. Właśnie dlatego właściwości wejściowe muszą być znane Angularowi, gdyż w przeciwnym razie nie wiedziałby, jak wykrywać zmiany. Angular ma dwa trybty wykrywania zmian:
* **Default** - Tryb domyślny, zmiany komponenu nadrzędnego pociągają za soboą konieczność sprawdzenia pod kątem zmian wszystkie jego dzieci
* **Tryb OnPush** - Informuje Angulara, że dany komponent wymaga sprawdzania pod kątem zmian tylko wtedy, gdy ulegnie zmianie jedno z wejść komponentu. To oznacza, że jeśli w komponencie nadrzędnym nie zostały zmienione dane, które równocześnie są wejściami komponenu OnPush to ten komponent nie wymaga ponownego renderowania. Oznacza to także, że jego komponenty potomne też nie wymagają renderowania. Do tego typy renderowania dobrze nadaje się komponent typy **display**, gdzie wszystkie dane przekazujemy przez wiązane wejściowe. Aby wygenerować komponent OnPush można skorzystać z generatora **VS CODE**

#### Komunikacja między komponentami
Wejścia są sposobem przesyłania danych w dół drzewa komponentów do potomków, a zdarzenia są sposobem przekazywania danych i powiadamiania w górę drzewa komponentów do rodzica.

#### Komunikacja do rodzica
Output  i EventEmitter należy importować z `@angular/core`
```typescript
import { Component, OnInit, Output, EventEmitter } from '@angular/core';
```
Komunikacja odbywa się zawsze z wykorzystaniem wiązania eventów. Przykład komponent rodzica odbiera notyfikacje od dziecka"
**Komponent dziecka** dostępnia punkt zaczepienia. Do tego punkt komponent rodzica przekazuje metodę obsługującą to zdarzenaie. Zdarzenie generuje również pakiet wiadomości. W tym przypadku nie przkazuje żadnych danych, a jest wykrozystywany tylko do powiadomienia o zdarzeniu - typ generyczny to null.

```typescript
export class NotifyParentComponent {
  @Output() notificationHandler = new EventEmitter<null>();

  notify() {
    this.notificationHandler.emit();
  }
}
```
**Parent Templete** Komponent rodzica przekazuje funkcje obsługującą zdarzenie

```html
<app-container>
    <app-notify-parent (notificationHandler)="parentHandler()"></app-notify-parent>
</app-container>
```

#### Dodawanie styli do komponentu

* Podejście globalne - style są definiowane w `style.css` lub jest to zewnętrzny `url` dodawny do `index.html`.
* Style związane z komponentem
* Przez wiązanie @Input

[Dynamiczne tworzenie komponentu]

## Angular Animation
#### Good links:
- [Docs](https://angular.io/guide/animations)

### Animations Artifacts:
- `trigger()` -  Jest to kontener na stany i przejścia oraz nadaje animacje nazwę, dzięki czemu można dołączyć ją do elementu DOM w HTML templete. Wykorzystując tą notację.
- `state()` - jest punktem końcowym przejścia, posiadjącym style. Po wykonanym przejściu style są zachowane.
- `transition()` - jest przejściem między stanami. Pozwaja na definiowanie styli pośrednich, które tworzą iluzję ruchu podczas animacji
- `animate()` - jest zbiorem stanów oraz przejśc

#### Stany:
- **Void state** - nie jest jeszcze w struktrzue DOM. Przykładowo jeżeli element jest utworzony ale nie jest dodany do drzewa DOM lub jeżeli został usunięty z DOMu.
- **Default (*) state** - jest domyślnym stanem, czyli tym które są zdefiniowane z wykłym CSSie. Jest oznaczany w kodzie za pomocą `*`
- **Custom state** - jest własnym zdefiniowanym stanem. Jest definiowany za pomocą funkcji `state()`

### Przykładowy komponent
```html
<div
  [@openClose]="isOpen ? 'open' : 'closed'"
  class="open-close-container"
  (click)="toggle()"
>
  <p>The box is now {{ isOpen ? 'Open' : 'Closed' }}!</p>
</div>

```
```typescript
@Component({
  ...
  animations: [
    trigger('openClose', [
      state(
        'open',
        style({
          height: '200px',
          opacity: 1,
          backgroundColor: 'yellow',
        })
      ),
      state(
        'closed',
        style({
          height: '100px',
          opacity: 0.5,
          backgroundColor: 'green',
        })
      ),
      transition('open => closed', [animate('1s')]),
      transition('closed => open', [animate('0.5s')]),
    ]),
  ],
})
export class AppComponent {
  isOpen = true;

  toggle() {
    this.isOpen = !this.isOpen;
  }
}
```
### Opis implementacji

- **trigger**
Animacja jest wykonywana lub wyzwalana, gdy wartość wyrażenia zmienia się w nowy stan. Gdzie stan jest zawsze nazwą stanu zdefiniowanego w triggerze.
```html
<div [@triggerName]="expression">...</div>
```
Trigger posiada dwa paramtery. Pierwszy to nazwa triggera, która ma zostać dołączona do HTML template. Drugi to tablica zawierając definicję wszsytkich stanów oraz przejść między stanami.
```typescript
	trigger('trigger-name', [
		state(),
		transition()
	])
```
- **state**
Stan posiada swoją nazwę  nazwę oraz związane z nimi style
```typescript
	state('state-name',
		 style()
	])
```
- **style**
Style są zdefiniowane w obiekcie JSON
```typescript
	style({
		property: 'property-value'	
	}
	])
```
- **transition**
Przyjmuje dwa argumenty. Pierwszy to przejście w postacji stringa, oraz drugi to tablica zawierająca animację. Nazwy stanów mogą być zdefiniowane w triggerze lub być stanami `void` lub default `*`
```typescript
	transition('void => *', [animate()]),
```
- **animate**
duration - czas drawnia
delay - opóźnienie
easing - [rodzaj przyśpieszenia](https://material.io/design/motion/speed.html#)
```typescript
	animate ('duration delay easing')
```


#### Sposób na tworzenie animacji wewnątrz dyrektywy
- [link do stacka](https://stackoverflow.com/questions/42878197/angular-2-how-to-set-animations-on-a-directive)
- [przykładowy program - stackbliz](https://stackblitz.com/github/zetsnotdead/ng-programmatic-animation-construction?file=src%2Fapp%2Ffade-in-out.directive.ts)


### From

### Modal window
Okno modalne  na specjanym typem okna dialogowego. Jest oknem nie pozwalającym na obsługę zdarzeń dotyczących pozostałych okien danej aplikacji. Wówczas żadane inne okna aplikacji nie reagują na działania użytkownika .

### Services

#### Links:
