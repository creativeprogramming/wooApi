# WooApi
Woocommerce API service with angular

##Supports angular version
- master - Angular 6
- v5 - Angular 5

#Supports woocommerce API version
- Supports V3 version

Tested with https protocol. Wordpress version 4.9.6 and WooCommerce version 3.4.1

## Api Doc
https://angular-studio.github.io/wooApi/

## Enable CORS
Add this code in function.php

```
add_action( 'init', 'nt_cors_enable' );

function nt_cors_enable() {
  header("Access-Control-Allow-Origin: " . get_http_origin());
  header("Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, DELETE");
  header("Access-Control-Allow-Credentials: true");
  header("Access-Control-Allow-Headers: Authorization, Content-Type");
  header("Access-Control-Expose-Headers: x-wc-totalpages, x-wc-total", false);
  if ( 'OPTIONS' == $_SERVER['REQUEST_METHOD'] ) {
    status_header(200);
    exit();
  }
}
```

## Setup instruction

- `yarn add ngx-wooapi@5.x.x` or `npm install --save ngx-wooapi@5.x.x`
- Add interceptor 

```
import {
  Injectable,
  // Injector
 } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
  HttpErrorResponse
} from '@angular/common/http';
// import { Router } from '@angular/router';
import { catchError } from 'rxjs/operators';
import { Observable } from 'rxjs';

// import { AuthService } from './auth.service';
import { environment } from '../environments/environment';

@Injectable()
export class AppInterceptor implements HttpInterceptor {

  constructor(
    // private injector: Injector,
    // private router: Router
  ) { }

  private includeWooAuth(url) {
    const wooAuth = `consumer_key=${environment.woocommerce.consumer_key}&consumer_secret=${environment.woocommerce.consumer_secret}`;
    const hasQuery = url.includes('?');
    let return_url = '';
    if (hasQuery) {
      return_url =  wooAuth;
    } else {
      return_url = '?' + wooAuth;
    }
    return return_url;
  }

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // const auth = this.injector.get(AuthService);
    const authRequest = request.clone({
      setHeaders: {
        // Authorization: `Bearer ${auth.getToken()}`
      },
      url: `${environment.origin}/${request.url}${this.includeWooAuth(request.url)}`
    });

    return next.handle(authRequest)
      .pipe(
        catchError(err => {
          if (err instanceof HttpErrorResponse && err.status === 0) {
            console.log('Check Your Internet Connection And Try again Later');
          } else if (err instanceof HttpErrorResponse && err.status === 401) {
            // auth.setToken(null);
            // this.router.navigate(['/', 'login']);
          }
          return Observable.throw(err);
        })
      );
  }
}

```

Add this code in your app.module.ts

```
 providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AppInterceptor,
      multi: true
    }
 ]

```
Add new property in environment.ts for angular webapp 

```
origin: 'https://domain/appwoo/wc-api/v3',
  woocommerce: {
    consumer_key:  'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    consumer_secret: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
  }
```

Add new providers in app.module 
```
import {
  WoocommerceProductsService,
  WoocommerceHelperService
} from 'ngx-wooapi';
```
Add these providers in providers array

```
providers: [
  WoocommerceProductsService,
  WoocommerceHelperService
]
```

Now use it in component

```
import {
  WoocommerceProductsService
} from 'ngx-wooapi';

constructor(
    private wooProducs: WoocommerceProductsService
  ) { }

  ngOnInit() {
    this.wooProducs.retrieveProducts().subscribe(response => {
      console.log(response);
    }, err => {
      console.log(err);
    });
  }

```

All done have fun :)