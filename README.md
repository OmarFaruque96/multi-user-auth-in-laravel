# multi-user-auth-in-laravel
step by step code for multi user authentication in laravel

#user-auth-command
#using Breeze (please take a backup for route)

- composer require laravel/breeze --dev
- php artisan breeze:install

#setup all pages now (login,reg,forgetpass,dashboard etc)
#multi-user-auth

- create role col in user database
- migrate

1. Open app/User.php file and add following code to our model:

    //code
  
    public function isAdmin() {
       return $this->role === 'admin';
    }

    public function isUser() {
       return $this->role === 'user';
    }
    
2. Open your terminal window again by being in laravel project root directory and run following commands:
  - php artisan make:middleware UserAuthenticated
  - php artisan make:middleware AdminAuthenticated
  
3. admin middle ware 

  namespace App\Http\Middleware;

    use Auth;
    use Closure;

    class AdminAuthenticated
    {
        public function handle($request, Closure $next)
        {
            if( Auth::check() )
            {
                // if user is not admin take him to his dashboard
                if ( Auth::user()->isUser() ) {
                     return redirect(route('user_dashboard'));
                }

                // allow admin to proceed with request
                else if ( Auth::user()->isAdmin() ) {
                     return $next($request);
                }
            }

            abort(404);  // for other user throw 404 error
        }
    }

4. user middleware

  namespace App\Http\Middleware;

    use Auth;
    use Closure;

    class UserAuthenticated
    {
        public function handle($request, Closure $next)
        {
            if( Auth::check() )
            {
                // if user admin take him to his dashboard
                if ( Auth::user()->isAdmin() ) {
                     return redirect(route('admin_dashboard'));
                }

                // allow user to proceed with request
                else if ( Auth::user()->isUser() ) {
                     return $next($request);
                }
            }

            abort(404);  // for other user throw 404 error
        }
    }

5. Open app/Http/Kernel.php file and add following two lines:

  namespace App\Http;

    use Illuminate\Foundation\Http\Kernel as HttpKernel;

    class Kernel extends HttpKernel
    {
        // some other functions

        protected $routeMiddleware = [
            // some other middlewares
            'admin' => \App\Http\Middleware\AdminAuthenticated::class,
            'user'  => \App\Http\Middleware\UserAuthenticated::class
        ];
    }

6. Adding middleware to routes

  
    Route::group(['middleware' => ['auth', 'user'], 'prefix' => 'user'], function () {
        Route::get('/', 'HomeController@index')->name('user_dashboard');
    });

    // admin protected routes
    Route::group(['middleware' => ['auth', 'admin'], 'prefix' => 'admin'], function () {
        Route::get('/', 'HomeController@index')->name('admin_dashboard');
    });

7. last change we have to make is in LoginController.

  namespace App\Http\Controllers\Auth;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\AuthenticatesUsers;

    class LoginController extends Controller
    {

        use AuthenticatesUsers;

        // some other functions go here

        protected function authenticated(Request $request, $user)
        {
            // to admin dashboard
            if($user->isAdmin()) {
                return redirect(route('admin_dashboard'));
            }

            // to user dashboard
            else if($user->isUser()) {
                return redirect(route('user_dashboard'));
            }

            abort(404);
        }
    }

#thats-it
