class: center, middle

# Front End Development
### With Examples from Lot Maintenance

---

# Agenda

1. Angular and TypeScript
2. Angular Components, Services and Modules
3. Templating in Angular
4. Unit Testing with Karma and Jasmine
5. E2E Testing with Protractor
6. RxJS
7. SCSS (Sass)
8. BEM
9. Gcm.UX Package

---

# Angular
- A frontend framework developed by Google
- V2+ had help from MS (written in TypeScript)
  - A ground-up rewrite after AngularJS

##### TypeScript
- Anders Hejlsberg, chief C# architect, involved in development
- A superset of ECMAScript 2015 (also called ECMAScript6)
  - Official name for JS version supported by modern browsers
- Adds static typing and compile-time type checking

---

# TypeScript Syntax
```typescript
export class Lot { // A class with a constructor

  constructor(balance: number) { // Constructor parameters with types
    this.balance = balance;
  }

  lotDate: string; // Public properties with types
  lotId: number;
  balance: number;
  lotDateFormatted?: any;
}

export class Routes { // A class without a constructor
  public static tasks = '/api/tasks'; // Static strings
  public static portfolios = '/api/portfolios';
  public static username = '/.auth/me';
  public static log = '/client/log';
}

export interface Column { // An interface
  name: string;
  field: string;
  sort: Sort;
}

export enum Sort { // An enum
  ASC = 'asc',
  DESC = 'desc'
}
```

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
- Shared singletons, single-purpose, stateful
- Abstract away state/functionality used by other components/services

##### Like components:
- Services are stateful
- Services are classes, with access modifiers
- Services have constructors, use constructor injection

##### Unlike components:
- Services not bound to DOM
- There should exist only one instance
- Meant to be injected (Injectable decorator)

The [docs](https://angular.io/guide/architecture-services) go into greater detail.
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

  constructor( // DI in constructor, invoked once in app lifecycle
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
- Think .csproj file in C#/.NET solution
- Makes Angular DI possible

#### Includes:
- Declarations: Component definitions
- Exports: Subset of declarations
- Imports: Dependencies
- Providers: Service definitions (global)
- Bootstrap: Used by AppModule only

#### Like services and components, just a class + decorator
Here's the [module section](https://angular.io/guide/architecture-modules) of the Angular tutorial.
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
- Data formatting (via pipes) and conditional rendering
- Iterating over collections

### Comparable to:
Other client-side/js templating libraries:
 - Handlebars.js
 - Mustache.js
 - Knockout.js data-binding

Server-side templating libraries:
 - Razor
 - Jinja2 (Python)

Read more in the [docs](https://angular.io/guide/template-syntax)
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

# Unit Testing
#### Karma
- A JavaScript test runner
- Runs a local web server, launches browsers specified in Karma config
  - Usually Chrome/Headless Chrome

##### [Karma Docs](https://karma-runner.github.io/latest/index.html)

#### Jasmine
- A BDD testing framework for JavaScript, but it doesn't use Gherkin
- Think Cucumber or Specflow

##### [Jasmine Docs](https://jasmine.github.io/api/3.3/global)

---
# Unit Test Syntax
```javascript
import { async, TestBed } from '@angular/core/testing';
import { HttpClientTestingModule,HttpTestingController,TestRequest } from '@angular/common/http/testing';
import { of } from 'rxjs';
import { ApiService } from '@lot-maintenance-ui-http/api.service';
import { portfolioMock } from '@lot-maintenance-ui-core/mocks/create-task.mock';
import { PortfolioService } from './portfolio.service';

describe('PortfolioService', () => {
  let service: PortfolioService; let apiService: ApiService;
  let httpMock: HttpTestingController; let request: TestRequest[];

  beforeEach(async(() => { // Think NUnit [SetUp] 
    TestBed.configureTestingModule({ // This should look like the @NgModule decorator
      imports: [  HttpClientTestingModule ], 
      providers: [ ApiService, PortfolioService ]
    })
    .compileComponents();

    apiService = TestBed.get(ApiService); 
    service = TestBed.get(PortfolioService);
    httpMock = TestBed.get(HttpTestingController);
    spyOn(apiService, 'getPortfolioList').and.returnValue((of(portfolioMock)));
    apiService.getPortfolioList().subscribe((list) => { expect(list).toEqual(portfolioMock); });
    request = httpMock.match(`/api/portfolios`);
    expect(request[0].request.method).toBe('GET');
  })); // End of beforeEach()

  it('should return portfolios', () => {
    expect(service.getPortfolioList()).toEqual([]); // Before API returns portfolios, list is empty
    request[0].flush(portfolioMock);
    expect(service.getPortfolioList()).toEqual(portfolioMock); // Afterwards, returns mock response
  });
});
```
---

# E2E Testing with Protractor
- Uses Node.js Selenium webdriver under the hood
  - WebDriverJS
- Automation of an actual browser
- Acceptance/Integration tests
- Looks a bit like Selenium/PageObjects

Read the Protractor [docs](http://www.protractortest.org/#/).

---

# Protractor Test Syntax
##### Page Object
```javascript
import { browser, by, element } from 'protractor';

export class AppPage {
    navigateTo() { return browser.get('/'); }
    getParagraphText() { return element(by.css('app-root h1')).getText(); }
}
```
##### Protractor Test
```javascript
import { AppPage } from './app.po';

describe('workspace-project App', () => {
  let page: AppPage;
  beforeEach(() => {  page = new AppPage(); });

  it('should display app title', () => {
    page.navigateTo();
    expect(page.getParagraphText()).toEqual('LOT MAINTENANCE');
  });
});
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

---
# SCSS (Sass)
- One of many CSS extension languages
- At build time, SCSS is transpiled into CSS using a pre-processor
- A bit like LESS
- Variables
- Nesting for child combinators or concatenation (with '&')
  - Because we're using BEM, we try to avoid child combinators
- Mixins
- Importing
- Built-in functions and operators

Here are the [docs](https://sass-lang.com/guide).
---

# SCSS Syntax
```scss
@import 'variables';

.app-header {
    height: 72px;
    text-align: center;
    background-color: $primary;

    &__content {
        @extend %content-wrap;
        position: relative;
    }

    &__title {
        color: #fff;
        font-size: 42px;
        font-family: "open_sanslight",sans-serif;
    }

    &__nav {
        position: absolute;
        right: 25px;
        top: 0;
        bottom: 0;
        height: 80%;
        margin: auto;
        display: block;
        &__item {
            cursor: pointer;
            background-color: desaturate(lighten($primary,13%),15%);

            &:hover{
                background-color: desaturate(lighten($primary,29%),22%);
            }
        }
    }
}
```

---
# (A)BEM

#### BEM is Block Element Modifier
- CSS can get messy, specificity is [confusing](https://www.w3.org/TR/selectors/#specificity).
- BEM is a naming convention and a methodology for defining styles
- All selectors limited to a single class
- No IDs, no combinators, no tag names
- All styles are of equivalent specificity
- .{BLOCK}__{ELEMENT}--{MODIFIER}

#### ABEM is a variant that uses separated modifiers
- .{BLOCK}__{ELEMENT} -{MODIFIER} -{MODIFIER} -{MODIFIER}
- Styles become less verbose
- We're not using all of ABEM, just the approach to modifiers

##### [BEM Docs](http://getbem.com/naming/)
##### [ABEM Article](https://css-tricks.com/abem-useful-adaptation-bem/)
---

# A Case for BEM

#### What color will the text be?
##### CSS
```scss
.parent#foo .child {
    color: green;
}
div.parent > .child {
    color: red;
}
#foo :not(#baz) {
    color: orange;
}
span.child {
    color: blue;
}
```
##### Affected HTML
```html
<div id="foo" class="parent">
  <span id="bar" class="child">What color am I?</span>
</div>
```
---

# ABEM Syntax
```scss
/* Allows us to express icons more flexibly, the following are all valid:
* <span class="icon -phone -large -bold"></span>
* <div class="icon -reply -xlarge"></div>
* <strong class="icon -popup"></strong>
*/
.icon {
    font-size: 11px;
    font-family: 'gcm-icon-fonts';
    font-style: normal;
    /* ... */
    &.-phone:before {content: "";} // The following are non-printable characters for icon font
    &.-envelope:before {content: "";}
    &.-popup:before {content: "";}
    &.-documents:before {content: "";}
    &.-fundscreener:before {content: "";}
    &.-reply:before {content: "\e610";} // Last two are unicode characters
    &.-prev:before {content: "\e60f";}
    
    &.-large {font-size: 16px;}
    &.-xlarge {
        font-size: 28px;
        padding-bottom: 4px;
    }
    &.-bold {font-weight: 600;}
}
```

---

# Gcm.UX Package/Style Guide
- Deployed as a KSS style guide and an npm package
  - Knyle Style Sheets produces a living style guide from CSS documentation

#### Deployment Process:
 - Transpile SCSS >> CSS
 - Run ```npm pack```, produces gzipped tarball of directory contents
 - Build KSS styleguide from CSS docs
 - Upload both package and styleguide to blob storage

##### Styleguide lives in Azure Storage, hosted as a static site [here](https://gcmux.z19.web.core.windows.net/).

---

# Recap:
#### Angular
 - Uses TypeScript - think JS with types
 - Components use @Component, two-way data binding to DOM
 - Services use @Injectable, and are singletons, single-purpose, and stateful
 - Modules handle importing, exporting
 - If you know Razor/Knockout, Angular templating is pretty similar
 - Jasmine is a BDD test framework that supports mocks/spies/assertions/etc.
 - *Jasmine* tests run on *Karma* test runner
 - Protractor is used for End-to-End tests
   - Uses Selenium, encourages PageObjects

#### RxJS
 - Data streams and observables
 - Asynchronous programming approaches necessary due to JS single-threadedness
   - Callbacks >> Promises >> RxJS/Observables
 - Think pub/sub

---

# Recap (Continued):

#### SCSS
 - A CSS extension language that needs to be pre-processed/transpiled
 - Provides nesting, inheritance, importing, variables, and built-in functions

#### (A)BEM
 - CSS can get messy
 - Styles are expressed as a single class
   - Unless they are modifying classes
 - There should be no competing styles/overrides or specificity concerns