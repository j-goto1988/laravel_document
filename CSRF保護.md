## CSRFリクエストの防止
アプリケーションによって管理されているアクティブなユーザーセッションごとにCSRF「トークン」を自動的に生成する。
このトークンは、認証済みユーザーが実際にアプリケーションへリクエストを行っているユーザーであることを確認するために使用される。
このトークンはユーザーのセッションに保存され、セッションが再生成されるたびに変更されるため、悪意のあるアプリケーションはこのトークンへアクセスできない。

現在のセッションのCSRFトークンには、リクエストのセッションまたはcsrf_tokenヘルパ関数を介してアクセスできる。
```PHP
use Illuminate\Http\Request;
Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

アプリケーションで"POST"、"PUT"、"PATCH"、"DELETE" HTMLフォームを定義するときはいつでも、CSRF保護ミドルウェアがリクエストを検証できるように、フォームに非表示のCSRF_tokenフィールドを含める必要がある。
便利なように、@csrf Bladeディレクティブを使用して、非表示のトークン入力フィールドを生成できる。
```PHP
<form method="POST" action="/profile">
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

webミドルウェアグループへデフォルトで含まれているApp\Http\Middleware\VerificationCsrfTokenミドルウェアは、リクエスト入力のトークンがセッションに保存されたトークンと一致するかを自動的に検証する。
この2トークンが一致すれば、認証済みユーザーがリクエストを開始したことがわかる。

一連のURIをCSRF保護から除外したい場合は、routes/web.phpファイル中で、App\Providers\RouteServiceProviderがすべてのルートへ適用するwebミドルウェアグループの外側に配置する必要がある。
VerifyCsrfTokenミドルウェアの$exceptプロパティにURIを追加してルートを除外することもできる。