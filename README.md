# OAuth Google API

- OAuth Protocole : used for authentification & autorisation
- we will use it to access Google API


# 1 — Install Laravel Socialite

```bash
composer require laravel/socialite
```

# 2 — Configure Google credentials

- Get client OAuth 2.0 from Google API Console.
https://console.cloud.google.com/projectselector2/apis/dashboard?hl=fr&inv=1&invt=Ab2XtA&supportedpurview=project

1. Go to Google Cloud Console.
2. Create a new project (or use an existing one).
3. Go to OAuth consent screen → Configure it.
4. Go to APIs & Services > Credentials:
  - Click Create Credentials > OAuth client ID
  - Choose Web Application
  - Set Authorized redirect URI to: http://localhost:8000/auth/google/callback
5. Copy the Client ID and Client Secret
6. Add these to your .env file:

```
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_REDIRECT_URI=http://localhost:8000/auth/google/callback
```

# 3 — Configure Socialite in config/services.php

```php
'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_REDIRECT_URI'),
],
```

# 4 Add routes in routes/web.php (Google callback route)

```php
use Laravel\Socialite\Facades\Socialite;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;
use App\Models\User;

Route::get('/auth/google/callback', function () {
    $googleUser = Socialite::driver('google')->stateless()->user();

    // ✅ 1. Check if the email exists
    if (!$googleUser->getEmail()) {
        abort(403, 'Google account has no email.');
    }

    // ✅ 2. Find or create the user
    $user = User::firstOrCreate([
        'email' => $googleUser->getEmail(),
    ], [
        'name' => $googleUser->getName(),
        'email_verified_at' => now(), // ✅ 3. Automatically mark email as verified
        'password' => Hash::make(Str::random(24)),
        'profile_photo_path' => $googleUser->getAvatar(), // ✅ 4. Save avatar URL (optional)
    ]);

    Auth::login($user, remember: true);

    return redirect()->intended('/dashboard');
});
```

🚨 **InUser Model : Alert : you must change 'email_verified_at' to be $fillable**

## Optional — Updating existing user's name or avatar

```php
if (!$user->wasRecentlyCreated) {
    $user->update([
        'name' => $googleUser->getName(),
        'profile_photo_path' => $googleUser->getAvatar(),
    ]);
}
```

# 5 Add a "Login with Google" button in your login/register Blade

- Example in resources/views/auth/login.blade.php or in your layout:

```html
<a href="{{ route('google.login') }}"
   class="inline-flex items-center px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700">
   <svg class="w-5 h-5 mr-2" viewBox="0 0 48 48"><path fill="#FFC107" d="..."/></svg>
   Login with Google
</a>
```

# Optional: Redirect only verified emails

- If you want to ensure the email is verified, you can check inside the callback:

```
if (!$googleUser->getEmail()) {
    abort(403, 'Google account has no email.');
}
```

- And you can also use:

```php
'email_verified_at' => now(), // mark the email as verified
```



