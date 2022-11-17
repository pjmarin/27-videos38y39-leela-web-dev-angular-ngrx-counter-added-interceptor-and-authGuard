En estos 2 videos hemos eliminado el almacenamiento del usuario en localStorage, ya que lo tenemos en la store de ngrx,
y de ahi es de donde cogemos ahora el token para estar autenticados en todo momento. Tambien hemos enviado las cookies
desde el front, y ya aparecen en la pestana application del chrome developer tools.

Hemos implementado AuthGuard como nuevo servicio y lo utilizamos en app-routing-module.ts en la ruta de los posts --> canActivate: [AuthGuard]

Hemos corregido el bug del funcionamiento del logout, el cual se deslogaba, y despues se volvia a logar solo, al intentar 
corregirlo se producian otros bugs, como llamadas multiples al server, al final hemos incorporado en los effects del login
el operador take con el valor 1 --> take(1) despues del exhaustMap, es decir hemos cambiado:

login$ = createEffect(() => {
    return this.actions$.pipe(
      ofType(loginStart),
      exhaustMap((action) => {
        return this.authService.login(action.email, action.password).pipe(
          map((data) => {
            this.store.dispatch(setLoadingSpinner({ status: false }));
            this.store.dispatch(setErrorMessage({ message: '' }));
            const user = this.authService.formatUser(data);
            // this.authService.setUserInLocalStorage(user); // antes guardabamos el user en localstorage, ahora lo guardamos directamente en la store (ngrx)
            this.authService.runTimeoutInterval(user);
            this.authService.setRefreshToken(data);
            return loginSuccess({ user, redirect: true });
          }),
          catchError(errResp => {
            this.store.dispatch(setLoadingSpinner({ status: false }));
            const errorMessage = this.authService.getErrorMessage(errResp.error?.error);
            return of(setErrorMessage({ message: errorMessage }));
          })
        );
      })
    );
  });

  por:

  login$ = createEffect(() => {
    return this.actions$.pipe(
      ofType(loginStart),
      exhaustMap((action) => {
        return this.authService.login(action.email, action.password).pipe(
          map((data) => {
            this.store.dispatch(setLoadingSpinner({ status: false }));
            this.store.dispatch(setErrorMessage({ message: '' }));
            const user = this.authService.formatUser(data);
            // this.authService.setUserInLocalStorage(user); // antes guardabamos el user en localstorage, ahora lo guardamos directamente en la store (ngrx)
            this.authService.runTimeoutInterval(user);
            this.authService.setRefreshToken(data);
            return loginSuccess({ user, redirect: true });
          }),
          catchError(errResp => {
            this.store.dispatch(setLoadingSpinner({ status: false }));
            const errorMessage = this.authService.getErrorMessage(errResp.error?.error);
            return of(setErrorMessage({ message: errorMessage }));
          })
        );
      }), take(1) // Fix for logout, avoids to re-login again automatically when click logout button
    );
  });
  
  En el video original tambien se corrige agregando take(1), pero en lugar de hacerlo en el effect del login, lo hace en el archivo de interceptor
