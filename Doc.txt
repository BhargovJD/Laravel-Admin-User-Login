1.laravel new app

2.php artisan serve

3.in .env set database info

4.Auth system:
	composer require laravel/ui
	php artisan ui:auth
		it will create some folder in resources/views folder	

5.Add Boostrap(layouts/app.blade.php)
	css:<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
	
	bundle css:<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>
	
	add this two in layouts/app.blade.php
	
	add margin 'form-group row m-3' to auth/login.blade.php and auth/register.blade.php

6.Before register a user:
	php artisan migrate

7.After log in sometimes Dropdown list button will not work.
	edit layouts/app.blade.php

		<li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                                {{ Auth::user()->name }}
                            </a>
                            <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
                              <li>
                                  <a class="dropdown-item" href="#">My Profile</a>
                                </li>
                              <li>
                                  <a class="dropdown-item" href="{{ route('logout') }} " onclick="event.preventDefault();
                                  document.getElementById('logout-form').submit();">
                     {{ __('Logout') }} </a>
                     <form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
                        @csrf
                    </form>
                                </li>
                            </ul>
                          </li>

8.Admin role

	Databases/migrations/create users table.php

		$table->tinyInteger('role_as')->default('0');

	we already migrated our database so change directly in database.
	add one column in users as as_role which is tiny int and default value is 0.

9.middleware
	php artisan make:middleware AdminMiddleware
	Middleware created in App/Http/Middleware

10.in AdminMiddleWare
	use Illuminate\Support\Facades\Auth;

	public function handle(Request $request, Closure $next)
    {
        if(Auth::check()){
            if(Auth::user()->role_as=="1"){
                return $next($request);
            }
            else{
                return redirect('/home')->with('status','Access Denied! as you are not as admin');
            }
        }

        else{
            return redirect('/home')->with('status','please login first');
        }
    }

11.In Kernal.php we have to register our middleware:
	protected $routeMiddleware = [
        'isAdmin' => \App\Http\Middleware\AdminMiddleware::class,
    ];

12. Check who is login using controller

	In App/http/auth/LoginController.php

	use Illuminate\Support\Facades\Auth;


		protected function authenticated()
    {
        if(Auth::user()->role_as=='1')
        {
            return redirect('dashboard')->with('status','welcome to your dashboard');
        }
        elseif(Auth::user()->role_as=='0')
        {
            return redirect('/')->with('status','logged in successfully');
        }
    }

13. Using controller through route in Routes/Web.php

	Route::middleware(['auth','isAdmin'])->group(function(){
    Route::get('/dashboard',function(){
        return 'This is admin';
    });
});
