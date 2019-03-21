class: center, middle

# Lot Maintenance Overview + Angular for ASP.NET Developers

---

# Agenda

1. Components, Services and Modules
2. Templating
3. RxJS

---

# Angular Components

- Components are the "building blocks" of Angular 2+ apps, bound to the DOM.

- Components are used for:
  - Rendering data/component state (Compoment >> DOM)
  - Listening for user input (DOM >> Component)

- There can be multiple instances of a component at any one time
  - State is scoped to instance
  - When component is disposed of, state is lost

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
  - Only invoked once

Unlike components:
- Services not bound to DOM
- There should exist only one instance
- Meant to be injected
---

# Service Syntax
