# Angular Auth Guard

### here discussing frontend auth system only

1. Generate new app using CLI
2. Add components and routing.
3. Develop registration UI and service.
4. Develop login UI and service.

### to create new app
```bash
$ ng <app name> --routing=true --style=scss
```
### to create Servier Service

add HttpClient to module.ts file
```javascript
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
// HTTP_INTERCEPTORS - future for interceptor token  
```

 create a service for authentication.   
note: service have add manually to module.ts providers.
```bash
$ ng g s services/auth
```
```javascript
// authentication servies
import { AuthenticationService } from 'src/app/services/authentication/authentication.service';
providers: [AuthenticationService],
```



```javascript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { environment } from 'src/environments/environment';
import { Router } from '@angular/router';
import { User } from 'src/app/models/user';

@Injectable({
  providedIn: 'root'
})
export class AuthenticationService {

  constructor(private http:HttpClient,
              private router: Router) { }


  userLogin(userLogin:User){       
    return this.http.post(environment.adminApi+'/user/login',
    {
      username: userLogin.username,
      password: userLogin.password
    });
  }

  loggedIn(){
    return !!localStorage.getItem('token');
  }

  userLogout(){
     localStorage.removeItem('token');
     this.router.navigate(['/login']);
  }

  getToken(){
    return localStorage.getItem('token');
  }

}
```

add AuthenticationService to login function  (login.component.ts)
```javascript
password:string;
username:string;
constructor(
  private _auth:AuthenticationService,
  private _router:Router
) { }


inputUserName(event):void{
  this.username= event.target.value;
}
inputPassword(event):void{
  this.password= event.target.value;
}


login() {
  this._auth.userLogin({ password:this.password, username: this.username}).subscribe(res=>{

    if(res && res['success']){
      localStorage.setItem('token', res['token'] );
      this._router.navigate(['/main/dashboard']);
      console.log("login successfully");
    }else{
      //this.toastr.error('user credentials not matched', 'Error');
      console.log("error :", res['data']);
    }

  },err=>{
    //this.toastr.error(err, 'Error');
    console.log("Error :", err);
  });

}
```

### create AuthGuard
```bash
$ ng g guard auth
$ ng g guard  services/authentication/auth
```

```javascript

import { Injectable } from '@angular/core';
import { Router, CanActivate  } from '@angular/router';
import { AuthenticationService } from 'src/app/services/authentication/authentication.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {

  constructor( private router: Router,
               private auth:AuthenticationService) { }

  canActivate(): boolean {
    if(this.auth.loggedIn()){
      return true;
    }else{
      console.log("redireact to login...");
      this.router.navigate(['/login']);
      return false;
    }
  }

}
```

guard have add manually to module.ts providers.
```javascript
import { AuthGuard } from 'src/app/services/authentication/auth.guard';
providers: [AuthenticationService, AuthGuard],
```
### Add to Routing
then add AuthGuard's canActivate to routing (app-routing.module.ts)
```javascript
const routes: Routes = [
  {
    path: 'login',
    component: LoginComponent
  },
  {
    path: 'main',
    component: MainComponent,
    canActivate:[AuthGuard],
    children: [
      { path: 'dashboard', component: DashboardComponent, pathMatch: 'full' },
      { path: 'buyers', component: BuyersComponent },
      { path: 'sellers', component: SellersComponent },
      { path: 'items', component: ItemsComponent },
    ]
  },
  {
    path: 'register',
    component: RegisterComponent
  },
  {
    path: '',
    redirectTo: '/login',
    pathMatch: 'full'
  },
  { path: '**', redirectTo: '' }
];
```
### HTTP INTERCEPTORS
create interceptor service
```bash
$ ng g s services/authentication/token-interceptor
```

add HttpClient and created token-interceptor to module.ts file
```javascript
import { /*HttpClientModule,*/ HTTP_INTERCEPTORS } from '@angular/common/http';


providers: [ /*AuthGuard, */
  {
    provide:HTTP_INTERCEPTORS,
    useClass:TokenInterceptorService,
    multi:true
  }
],
```
token-interceptor.service.ts
````javascript
import { Injectable, Injector } from '@angular/core';
import { HttpInterceptor } from '@angular/common/http';
import { AuthenticationService } from 'src/app/services/authentication/authentication.service';


@Injectable({
  providedIn: 'root'
})
export class TokenInterceptorService implements HttpInterceptor {

  constructor(private injector: Injector) { }

  intercept(req, next){
    const auth = this.injector.get(AuthenticationService);
    const tokenizedReq = req.clone({
      setHeaders:{
         Authorization: `Bearer ${auth.getToken()}`
      }
    })
    return next.handle(tokenizedReq);
  }
}
```
