## 基礎
urlヘルパは、アプリケーションの任意のURLを生成するために使用する。
生成したURLは、アプリケーションが処理している現在のリクエストのスキーム(HTTPまたはHTTPS)とホストを自動的に使用する。
```PHP
$post = App\Models\Post::find(1);
echo url("/posts/{$post->id}");
// http://example.com/posts/1
```

urlヘルパにパスを指定しないと、Illuminate\Routing\UrlGeneratorインスタンスが返され、現在のURLに関する情報へアクセスできる。
```PHP
// クエリ文字列を除いた現在のURL
echo url()->current();
// クエリ文字列を含んだ現在のURL
echo url()->full();
// 直前のリクエストの完全なURL
echo url()->previous();
```

こうしたメソッドには、URLファサードを使用してもアクセスできる。
```PHP
use Illuminate\Support\Facades\URL;
echo URL::current();
```


## 名前付きルートのURL
routeヘルパは、名前付きルートへのURLを生成するためにも使用できる。
名前付きルートを使用すると、ルートで定義する実際のURLと結合せずにURLを生成できる。
したがって、ルートのURLが変更された場合でも、route関数の呼び出しを変更する必要はない。
たとえば、アプリケーションに次のように定義されたルートが含まれているとする。
```PHP
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```

このルートへのURLを生成するには、次のようにrouteヘルパを使用する。
```PHP
echo route('post.show', ['post' => 1]);
// http://example.com/post/1
```

routeヘルパを使用して、複数のパラメーターを持つルートのURLを生成することもできる。
```PHP
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    // ...
})->name('comment.show');
echo route('comment.show', ['post' => 1, 'comment' => 3]);
// http://example.com/post/1/comment/3
```

ルートの定義パラメータに対応しない過剰な配列要素は、URLのクエリ文字列として追加される。
```PHP
echo route('post.show', ['post' => 1, 'search' => 'rocket']);
// http://example.com/post/1?search=rocket
```

パラメータ値としてEloquentモデルを渡せる。
routeヘルパは、モデルのルートキーを自動的に抽出する。
```PHP
echo route('post.show', ['post' => $post]);
```

Laravelでは名前付きルートに対し、簡単に「署名付きURL」を作成できる。
このURLは「署名」ハッシュをクエリ文字列として付加し、作成されてからそのURLが変更されていないかをLaravelで確認できるようにする。
署名付きURLは公にアクセスさせるルートではあるが、URL操作に対する保護レイヤが必要な場合とくに便利。

名前付きルートに対し署名URLを作成するには、URLファサードのsignedRouteメソッドを使用する。
```PHP
use Illuminate\Support\Facades\URL;
return URL::signedRoute('unsubscribe', ['user' => 1]);
```

指定する時間が経過すると期限切れになる一時的な署名付きルートURLを生成する場合は、temporarySignedRouteメソッドを使用する。
Laravelが一時的な署名付きルートURLを検証するとき、署名付きURLにエンコードされている有効期限のタイムスタンプが経過していないことを確認する。
```PHP
use Illuminate\Support\Facades\URL;
return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

受信リクエストに有効な署名があるかどうかを確認するには、受信したIlluminate\Http\RequestインスタンスでhasValidSignatureメソッドを呼び出す。
```PHP
use Illuminate\Http\Request;
Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }
    // ...
})->name('unsubscribe');
```

クライアントサイドのペジネーションなど、アプリケーションのフロントエンドが署名付きURLにデータを追加することを許可する必要が起きる場合は、hasValidSignatureWhileIgnoringメソッドを用いて、署名付きURLを検証する際に無視すべきリクエストクエリパラメータを指定する。
パラメータの無視を許すと、誰でもリクエストのそのパラメータを変更できる点に注意。
```PHP
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```

受信リクエストインスタンスを使って署名付きURLを検証する代わりに、Illuminate\Routing\Middleware\ValidateSignatureミドルウェアをルートへ指定することもできる。
まだ割り当てていない場合は、HTTPカーネルの$middlewareAliases配列へ、このミドルウェアのエイリアスを割り当てる。
```PHP
/**
 * アプリケーションのミドルウェアのエイリアス
 *
 * ルートとグループへミドルウェアを便利に割り付けるため使用するエイリアス
 *
 * @var array<string, class-string|string>
 */
protected $middlewareAliases = [
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
];
```

ミドルウェアをカーネルに登録したら、それをルートにアタッチできる。
受信リクエストに有効な署名がない場合、ミドルウェアは自動的に403HTTPレスポンスを返す。
```PHP
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

期限切れになった署名付きURLを訪問すると、403 HTTPステータスコードの汎用エラーページが表示される。
例外ハンドラでInvalidSignatureException例外のカスタム"renderable"クロージャを定義することにより、この動作をカスタマイズできる。
このクロージャはHTTPレスポンスを返す必要がある。
```PHP
use Illuminate\Routing\Exceptions\InvalidSignatureException;
/**
 * アプリケーションの例外処理コールバックの登録
 */
public function register(): void
{
    $this->renderable(function (InvalidSignatureException $e) {
        return response()->view('error.link-expired', [], 403);
    });
}
```


## コントローラアクションのURL
action関数は、指定するコントローラアクションに対するURLを生成する。
```PHP
use App\Http\Controllers\HomeController;
$url = action([HomeController::class, 'index']);
```

コントローラメソッドがルートパラメータを受け入れる場合、関数の2番目の引数としてルートパラメータの連想配列を渡せる。
```PHP
$url = action([UserController::class, 'profile'], ['id' => 1]);
```


## デフォルト値
アプリケーションにより、特定のURLパラメータのデフォルト値をリクエスト全体で指定したい場合もある。
たとえば、多くのルートで{locale}パラメータを定義している。
```PHP
Route::get('/{locale}/posts', function () {
    // ...
})->name('post.index');
```

毎回routeヘルパを呼び出すごとに、localeをいつも指定するのは厄介。
そのため、現在のリクエストの間、常に適用されるこのパラメートのデフォルト値は、URL::defaultsメソッドを使用し定義できる。
現在のリクエストでアクセスできるように、ルートミドルウェアから、このメソッドを呼び出す。
```PHP
namespace App\Http\Middleware;
use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;
use Symfony\Component\HttpFoundation\Response;
class SetDefaultLocaleForUrls
{
    /**
     * 受信リクエストの処理
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```
一度localeパラメータに対するデフォルト値をセットしたら、routeヘルパを使いURLを生成する時に、値を渡す必要はない。

URLのデフォルト値を設定すると、Laravelの暗黙的なモデルバインディングの処理を妨げる可能性がある。
したがって、URLのデフォルトをLaravel自身のSubstituteBindingsミドルウェアの前に実行するよう設定するため、ミドルウェアの優先度を設定する必要がある。
それには、アプリケーションのHTTPカーネルの$middlewarePriorityプロパティ内にあるSubstituteBindingsミドルウェアの前にミドルウェアを確実に設置する。

$middlewarePriorityプロパティはIlluminate\Foundation\Http\Kernelベースクラスで定義されている。
変更するにはその定義をこのクラスからコピーし、アプリケーションのHTTPカーネルでオーバーライトする。
```PHP
/**
 * ミドルウェアの優先リスト
 *
 * この指定により、グローバルではないミドルウェアは常にこの順番になります。
 *
 * @var array
 */
protected $middlewarePriority = [
    // ...
     \App\Http\Middleware\SetDefaultLocaleForUrls::class,
     \Illuminate\Routing\Middleware\SubstituteBindings::class,
     // ...
];
```