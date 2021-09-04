# angular-routeswithguards

employee.ts (parent)

```js
export interface IEmployee {
  id: number;
  code: string;
  name: string;
  salary: number;
  starRating: number;
}
```

employee.service.ts

```js
import { HttpErrorResponse } from '@angular/common/http';
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable, throwError } from 'rxjs';
import { catchError, tap } from 'rxjs/operators';
import { IEmployee } from './employee';

@Injectable({
  providedIn: 'root'
})
export class EmployeeService {
  constructor(private http: HttpClient) {}

  getEmployees(): Observable<IEmployee[]> {
    return this.http
      .get<IEmployee[]>('https://609e2e9f33eed80017957f8d.mockapi.io/employee')
      .pipe(
        tap(data => console.log('All', JSON.stringify(data))),
        catchError(this.handleError)
      );
  }

  private handleError(err: HttpErrorResponse) {
    let errorMessage = '';
    if (err.error instanceof ErrorEvent) {
      errorMessage = `An Error Occured:${err.error.message}`;
    } else {
      errorMessage = `Server returned code:${err.status}, error message is:${
        err.message
      }`;
    }
    console.log(errorMessage);
    return throwError(errorMessage);
  }
}
```

employee-component.component.ts (parent)

```js
import { Component, OnDestroy, OnInit } from '@angular/core';
import { Subscription } from 'rxjs';
import { IEmployee } from './employee';
import { EmployeeService } from './employee.service';

@Component({
  selector: 'app-employee-component',
  templateUrl: './employee-component.component.html',
  styleUrls: ['./employee-component.component.css']
})
export class EmployeeComponentComponent implements OnInit, OnDestroy {
  sub!: Subscription;
  private _listFilter: string = '';
  private errorMessage: string = '';
  filteredEmployees: IEmployee[] = [];
  employees: IEmployee[] = [];

  constructor(private employeeService: EmployeeService) {}

  //ALTERNATIVE CODE OF CONSTRUCTOR
  // private employeeService;
  // constructor(employeeService: EmployeeService) {
  //   this.employeeService = employeeService;
  // }

  ngOnInit() {
    this.sub = this.employeeService.getEmployees().subscribe({
      next: employees => {
        this.employees = employees;
        this.filteredEmployees = this.employees;
      },
      error: err => (this.errorMessage = err)
    });
  }

  get listFilter(): string {
    return this._listFilter;
  }

  set listFilter(value: string) {
    this._listFilter = value;
    console.log('In Setter:', value);
    this.filteredEmployees = this.performFilter(value);
  }

  performFilter(filterBy: string): IEmployee[] {
    filterBy = filterBy.toLocaleLowerCase();
    return this.employees.filter((employee: IEmployee) =>
      employee.name.toLocaleLowerCase().includes(filterBy)
    );
  }

  ngOnDestroy() {
    this.sub.unsubscribe();
  }
}
```
employee-component.component.html (parent)

```html
<div>
  <div>
    <input type="text" [(ngModel)]="listFilter" />
  </div>
  <br />
  <div>
    Filter By: {{listFilter}}
  </div>
  <div>
    <table>
      <thead>
        <td>Id</td>
        <td>Code</td>
        <td>Name</td>
        <td>Salary</td>
        <td>Rating</td>
      </thead>
      <tr *ngFor="let employee of filteredEmployees">
        <td>{{employee.id}}</td>
        <td>{{employee.code}}</td>
        <td>
          <a [routerLink]="['/employees',employee.id]">
            {{employee.name}}
          </a>
        </td>
        <td>{{employee.salary}}</td>
        <td>
          <app-star [rating]="employee.starRating"></app-star>
        </td>
      </tr>
    </table>
  </div>
</div>
```
star.component.ts (child)

```js
import { Component, Input, OnChanges, OnInit } from '@angular/core';

@Component({
  selector: 'app-star',
  templateUrl: './star.component.html',
  styleUrls: ['./star.component.css']
})
export class StarComponent implements OnChanges {
  constructor() {}

  @Input() rating: number = 0;
  cropWidth: number = 75;

  ngOnChanges(): void {
    this.cropWidth = this.rating * (75 / 5);
  }
}
```

star.component.html (child)

```html
<div class="crop" [style.width.px]="cropWidth" title="rating">
  <div style="width:75px">
    <span class="fa fa-star"></span>
    <span class="fa fa-star"></span>
    <span class="fa fa-star"></span>
    <span class="fa fa-star"></span>
    <span class="fa fa-star"></span>
  </div>
</div>
```

star.component.css (child)

```css
.crop {
  overflow: hidden;
}
div {
  cursor: pointer;
}
```

employeedetail-component.component.ts

```js
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';

@Component({
  templateUrl: './employeedetail-component.component.html',
  styleUrls: ['./employeedetail-component.component.css']
})
export class EmployeedetailComponentComponent implements OnInit {
  pageTitle: string = 'Employee Detail';

  constructor(private route: ActivatedRoute, private router: Router) {}

  ngOnInit() {
    const id = Number(this.route.snapshot.paramMap.get('id'));
    this.pageTitle += `: ${id}`;
  }

  onBack(): void {
    this.router.navigate(['/employees']);
  }
}
```

employeedetail-component.component.html

```html
<div class="card">
  <div class="card-header">
    {{pageTitle}}
  </div>
  <div class="card-footer">
    <button class="btn btn-outline-secondary" (click)='onBack()'>
      <i class="fa fa-chevron-left"></i>Back
    </button>
  </div>
</div>
```

employeedetails.guard.ts

```js
import { Injectable } from '@angular/core';
import {
  CanActivate,
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  Router
} from '@angular/router';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class EmployeedetailGuard implements CanActivate {
  constructor(private router: Router) {}

  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean> | Promise<boolean> | boolean {
    const id = Number(next.paramMap.get('id'));
    if (isNaN(id) || id < 1) {
      alert('Invlaid Employee Id');
      this.router.navigate(['/employees']);
      return false;
    }

    return true;
  }
}
```

welcome-component.component.ts

```js
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-welcome-component',
  templateUrl: './welcome-component.component.html',
  styleUrls: ['./welcome-component.component.css']
})
export class WelcomeComponentComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

}
```

welcome-component.component.html

```html
<div>
  Welcome Works!
</div>
```


style.css

Install bootstrap font-awesome and import in style.css

```css
@import '~bootstrap/dist/css/bootstrap.min.css';
@import 'font-awesome/css/font-awesome.min.css';
```

app.module.ts

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';

import { AppComponent } from './app.component';
import { EmployeeComponentComponent } from './employee-component/employee-component.component';
import { StarComponent } from './shared/star/star.component';
import { HttpClientModule } from '@angular/common/http';
import { RouterModule } from '@angular/router';
import { EmployeedetailComponentComponent } from './employee-component/employeedetail-component/employeedetail-component.component';
import { WelcomeComponentComponent } from './welcome-component/welcome-component.component';
import { EmployeedetailGuard } from './employee-component/employeedetail-component/employeedetail.guard';

@NgModule({
  imports: [
    BrowserModule,
    FormsModule,
    HttpClientModule,
    RouterModule.forRoot([
      { path: 'employees', component: EmployeeComponentComponent },
      {
        path: 'employees/:id',
        component: EmployeedetailComponentComponent,
        canActivate: [EmployeedetailGuard]
      },
      { path: 'welcome', component: WelcomeComponentComponent },
      { path: '', redirectTo: 'welcome', pathMatch: 'full' },
      { path: '**', redirectTo: 'welcome', pathMatch: 'full' }
    ])
  ],
  declarations: [
    AppComponent,
    EmployeeComponentComponent,
    StarComponent,
    WelcomeComponentComponent,
    EmployeedetailComponentComponent
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

app.component.html

```html
<nav class='navbar navbar-expand navbar-light bg-light'>
  <a class='navbar-brand'>{{pageTitle}}</a>
  <ul class='nav nav-pills'>
    <li>
      <a class='nav-link' [routerLink]="['/welcome']">Home</a>
    <li>
    <li>
      <a class='nav-link' [routerLink]="['/employees']">Employee List</a>
    <li>

      <!-- <li>
  <a routerLink="/welcome">Home</a>
<li>
<li>
  <a routerLink="/employees">Employee List</a>
<li> -->
  </ul>
</nav>
<div class='container'>
  <router-outlet></router-outlet>
</div>
```
![image](https://user-images.githubusercontent.com/38757471/132080442-664d1f1d-2e42-47ac-88bc-984236beeab9.png)

![image](https://user-images.githubusercontent.com/38757471/132081691-7cad6e66-6fb3-4da0-82fb-b3d149bcfee0.png)






