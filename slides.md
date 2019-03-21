class: center, middle

# Lot Maintenance Overview + Angular for ASP.NET Developers

---

# Agenda

1. Angular Components, Services and Modules
2. Templating in Angular
3. RxJS
4. SCSS (Sass)

---

# Angular Components

- Components are the "building blocks" of Angular 2+ apps
- They are bound to the DOM
- They can be stateful

#### Components are used for:
- Rendering data/component state (Compoment >> DOM)
- Listening for user input (DOM >> Component)

#### Multiple Instances
- State is scoped to instance
- When component is disposed of, state is lost

Here are the [docs](https://angular.io/guide/architecture-components).
---

# Component Syntax

```javascript
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { debounceTime, distinctUntilChanged, map } from 'rxjs/operators';
import { PortfolioService } from 'portfolio/portfolio.service';
import { Portfolio } from '@lot-maintenance-ui-interfaces/portfolio.interface';

@Component({ // The Component decorator, required for all component classes
  selector: 'app-create-task-modal', // Used to find matching DOM element(s)
  templateUrl: './create-task.component.html', // The template used to render the component
  styleUrls: ['./create-task.component.scss'], // Styles for this component, with scoping
})

export class CreateTaskModalComponent {
  public portfolio: any;
  // Any private constructor argument will be injected and become a private property of the component
  constructor( private portfolioService: PortfolioService, ) { } 

  private getPortfolioList() { 
      return this.portfolioService.getPortfolioList(); // injected service now referenced as property
    } 

  // This is a public method, which can be invoked from the template/another component
  public containsTerm(item: string, term: string): boolean { 
      return item.toLowerCase().indexOf(term.toLowerCase()) > -1; }

  public filterPortfolioItems(term: string, list: Portfolio[]): Portfolio[] {
    return list.filter(item => this.containsTerm(item.portfolioAcronym, term)).slice(0, 10);
  }

  public portfolioSearch = (text$: Observable<string>): Observable<Portfolio[]> =>
    text$.pipe(debounceTime(200),distinctUntilChanged(),
      map(term => this.filterPortfolioItems(term, this.getPortfolioList()))
    )
} 
```

---

# Angular Services
- Singleton, single-purpose, stateful
- Services exist to abstract away shared state and functionality

Like components:
- Services are stateful
- Services are classes, with access modifiers
- Services have constructors, use constructor injection

Unlike components:
- Services not bound to DOM
- There should exist only one instance
- Meant to be injected (Injectable decorator)

Here are the [docs](https://angular.io/guide/architecture-services).
---

# Service Syntax
```javascript
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { ApiService } from '@lot-maintenance-ui-http/api.service';
import { Portfolio } from '@lot-maintenance-ui-interfaces/portfolio.interface';

@Injectable() // Injectable decorator, services are injected not bound to DOM
export class PortfolioService {
  private portfolioSubject: BehaviorSubject<Portfolio[]>;

  constructor(
    private apiService: ApiService,
  ) {
    this.portfolioSubject = new BehaviorSubject([]); // this is all RxJS
    this.apiService.getPortfolioList().subscribe((list: Portfolio[]) => {
      this.portfolioSubject.next(list);
    });
  }
  // Class properties/methods are public unless otherwise specified
  getPortfolioList(): Portfolio[] { 
    return this.portfolioSubject.value;
  }
}
```

---

# Angular Modules
Think namespaces or discrete projects in a C#/.NET solution

Includes:
- Declarations: Component definitions
- Exports: Subset of declarations
- Imports: Dependencies
- Providers: Service definitions (global)
- Bootstrap: Used by AppModule only

Like services and components, just a class + decorator
---

# Module Syntax
```javascript
@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    NotificationContainerComponent,
    TasksComponent,
    TaskDetailsComponent,
    CreateTaskModalComponent,
    TransactionsModalComponent,
    LoaderComponent,
    ConfirmationModalComponent,
  ],
  imports: [
    BrowserModule.withServerTransition({ appId: 'ng-cli-universal' }),
    HttpClientModule,
    FormsModule,
    ...
    AppRoutingModule
  ],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: LoaderInterceptorService,
      multi: true
    },
  ],
  bootstrap: [AppComponent],
  entryComponents: [
    CreateTaskModalComponent,
    ConfirmationModalComponent,
    TransactionsModalComponent,
  ]
})
export class AppModule { }
```

---

# Angular Templates 

Handles:
- Two-way model binding
- Data formatting (via pipes)
- Conditional rendering
- Iterating over collections
- Read more in the [docs](https://angular.io/guide/template-syntax)

### Comparable to:
Other client-side/js templating libraries:
 - Handlebars.js
 - Mustache.js
 - Knockout.js data-binding

Server-side templating libraries:
 - Razor
 - Jinja2 (Python)

---

# Template Syntax

```html
<div class="tasks container">
  <div class="row button-wrapper">
    <button class="btn btn-primary" (click)="openCreateModal()">Create</button>
  </div>
  <table class="table" *ngIf="tasks">
    <thead>
    <tr>
      <th *ngFor="let column of columns" class="table-header__col -sortable" 
        (click)="sortTable(column)" [ngClass]="getSortClass(column.sort)">
        {{column.name}}
      </th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let task of tasks" (click)="goToTaskDetails(task.taskId)" class="tasks-row">
      <td>{{ task.asOfDate }}</td>
      <td>{{ task.portfolio?.portfolioAcronym }}</td>
      <td>{{ task.investment?.investmentName }}</td>
      <td>{{ task.status }}</td>
      <td>{{ task.owner }}</td>
      <td>{{ task.reviewer }}</td>
    </tr>
    </tbody>
  </table>  
</div>
```
---

# RxJS
- RxJS is Reactive Extensions for JS, a reactive programming library.
  - Asynchronous operations as observable data streams.
  - Consumers of streams observe/react to propagated changes.
  - Think "pub/sub".


### Some background:
 - JS Runtime Environment single-threaded, no concurrent execution
   - Well, now there are Web Workers
 - Asynchronous programming to avoid blocking
 - Callbacks >> Promises >> Observables/RxJS
 - JavaScript also supports async/await now

Learn more [here](https://rxjs.dev/)

#

# Recap:
#### Angular
 - Components use @Component, two-way data binding to DOM
 - Services use @Injectable, they are singleton, single-purpose, and stateful
 - If you know Razor/Knockout, Angular templating is pretty similar