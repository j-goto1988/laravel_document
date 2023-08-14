## 概要
すべてのBladeテンプレートはプレーンなPHPコードにコンパイルされ、変更されるまでキャッシュされる。
Bladeテンプレートファイルは.blade.phpファイル拡張子を使用し、通常はresources/viewsディレクトリに保存する。

Bladeビューは、グローバルなviewヘルパを使用してルートまたはコントローラから返す。
データをviewヘルパの2番目の引数を使用してBladeビューに渡せる。
```PHP
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

## データの表示
変数を中括弧で囲むことにより、Bladeビューに渡すデータを表示できる。
```PHP
Hello, {{ $name }}.
```

PHP関数の結果をエコーすることもできる。
```PHP
The current UNIX timestamp is {{ time() }}.
```

デフォルトのBlade(およびLaraveleヘルパ)はHTMLエンティティをダブルエンコードする。
ダブルエンコーディングを無効にする場合は、AppServiceProviderのbootメソッドからBlade::withoutDoubleEncodingメソッドを呼び出す。
```PHP
namespace App\Providers;
use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;
class AppServiceProvider extends ServiceProvider
{
    /**
     * 全アプリケーションサービスの初期起動処理
     */
    public function boot(): void
    {
        Blade::withoutDoubleEncoding();
    }
}
```

データをエスケープしたくない場合は、次の構文を使用する。
```PHP
Hello, {!! $name !!}.
```

多くのJavaScriptフレームワークでも「中括弧」を使用して、特定の式をブラウザに表示することを示しているため、@記号を使用して式をそのままにしておくように、Bladeレンダリングエンジンへ通知できる。
```PHP
Hello, @{{ name }}.
```
この例の@記号はBladeが削除する。
{{ name }}式はBladeエンジンによって変更されないので、JavaScriptフレームワークでレンダリングできる。

@記号は、Bladeディレクティブをエスケープするためにも使用できる。
```PHP
{{-- Bladeテンプレート --}}
@@if()
<!-- HTML出力 -->
@if()
```

JavaScript変数を初期化するために、配列をJSONとしてレンダリングする目的でビューに配列を渡す場合がある。
```PHP
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

Illuminate\Support\Js::fromメソッドディレクティブが使える。
fromメソッドは、PHPのjson_encode関数と同じ引数を受け入れるが、取得結果のJSONはHTMLクオートの中へ含められるよう適切にエスケープされていることを保証する。
fromメソッドは、与えたオブジェクトや配列を有効なJavaScriptオブジェクトに変換するJSON.parse JavaScript文を文字列として返す。
```PHP
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

最新バージョンのLaravelアプリケーション・スケルトンには、Jsファサードが含まれており、Bladeテンプレート内でこの機能に簡単にアクセスできるようになっている。
```PHP
<script>
    var app = {{ Js::from($array) }};
</script>
```

テンプレートの大部分でJavaScript変数を表示している場合は、HTMLを@verbatimディレクティブでラップして、各Bladeエコーステートメントの前に@記号を付ける必要がないようにできる。
```PHP
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## Bladeディレクティブ
一般的なPHP制御構造への便利なショートカットも提供している。

@if、@elseif、@else、@endifディレクティブを使用してifステートメントを作成できる。
```PHP
@if (count($records) === 1)
    １レコードあります。
@elseif (count($records) > 1)
    複数レコードあります。
@else
    レコードがありません。
@endif
```

@unlessディレクティブも提供している。
```PHP
@unless (Auth::check())
    あなたはサインインしていません。
@endunless
```

@issetおよび@emptyディレクティブをそれぞれのPHP関数の便利なショートカットとして使用できる。
```PHP
@isset($records)
    // $recordsが定義済みで、NULLではない…
@endisset
@empty($records)
    // $recordsは「空」だ…
@endempty
```

@authおよび@guestディレクティブを使用すると、現在のユーザーが認証済みであるか、ゲストであるかを簡潔に判断できる。
```PHP
@auth
    // ユーザーは認証済み…
@endauth
@guest
    // ユーザーは認証されていない…
@endguest
```

@authおよび@guestディレクティブを使用するときにチェックする必要がある認証ガードを指定できる。
```PHP
@auth('admin')
    // ユーザーは認証済み…
@endauth
@guest('admin')
    // ユーザーは認証されていない…
@endguest
```

@productionディレクティブを使用して、アプリケーションが本番環境で実行されているかを確認できる。
```PHP
@production
    // Production限定コンテンツ…
@endproduction
```

@envディレクティブを使用して、アプリケーションが特定の環境で実行されているかどうかを判断できる。
```PHP
@env('staging')
    // アプリケーションは"staging"で動作している…
@endenv
@env(['staging', 'production'])
    // アプリケーションは"staging"か"production"で動作している…
@endenv
```

@hasSectionディレクティブを使用して、テンプレート継承セクションにコンテンツがあるかどうかを判断できる。
```PHP
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>
    <div class="clearfix"></div>
@endif
```

sectionMissingディレクティブを使用して、セクションにコンテンツがないかどうかを判断できる。
```PHP
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

Switchステートメントは、@switch、@case、@break、@default、@endswitchディレクティブを使用して作成できる。
```PHP
@switch($i)
    @case(1)
        最初のケース…
        @break
    @case(2)
        ２番めのケース…
        @break
    @default
        デフォルトのケース…
@endswitch
```
PHPのループ構造を操作するための簡単なディレクティブを提供する。
```PHP
@for ($i = 0; $i < 10; $i++)
    現在の値は、{{ $i }}
@endfor
@foreach ($users as $user)
    <p>このユーザーは：{{ $user->id }}</p>
@endforeach
@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>ユーザーはいません。</p>
@endforelse
@while (true)
    <p>無限ループ中です。</p>
@endwhile
```

ループを使用する場合は、@continueおよび@breakディレクティブを使用して、現在の反復をスキップするか、ループを終了することもできる。
```PHP
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif
    <li>{{ $user->name }}</li>
    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

ディレクティブ宣言に継続条件または中断条件を含めることもできる。
```PHP
@foreach ($users as $user)
    @continue($user->type == 1)
    <li>{{ $user->name }}</li>
    @break($user->number == 5)
@endforeach
```

foreachループの反復処理中、ループの内部では$loop変数を利用できる。
この変数により、現在のループのインデックスや、ループの最初の反復なのか最後の反復なのか、といった便利な情報にアクセスすることができる。
```PHP
@foreach ($users as $user)
    @if ($loop->first)
        これが最初の繰り返しです。
    @endif
    @if ($loop->last)
        これが最後の繰り返しです。
    @endif
    <p>このユーザーは：{{ $user->id }}</p>
@endforeach
```

ネストしたループ内にいる場合は、parentプロパティを介して親ループの$loop変数にアクセスできる。
```PHP
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            これは親ループの最初の繰り返しです。
        @endif
    @endforeach
@endforeach
```

@classディレクティブは、CSSのクラス文字列を条件付きでコンパイルする。
このディレクティブは、クラスの配列を受け取る。
配列のキーには、追加したいクラスが入り、値は論理値。
配列のキーが数字の場合は、レンダリングするクラスリストへ常に取り込む。
```PHP
@php
    $isActive = false;
    $hasError = true;
@endphp
<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>
<span class="p-4 text-gray-500 bg-red"></span>
```

@styleディレクティブは、条件付きでHTML要素へインラインのCSSスタイルを追加するために使用する。
```PHP
@php
    $isActive = true;
@endphp
<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>
<span style="background-color: red; font-weight: bold;"></span>
```

指定したHTMLのチェックボックス入力が"checked"であることを表す、@checkedディレクティブも使用できる。
このディレクティブは、指定条件がtrueと評価された場合、checkedをechoする。
```PHP
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

@selectedディレクティブは、特定のセレクトオプションが"selected"であることを表すために使用する。
```PHP
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

@disabledディレクティブは、指定要素が"disabled"であることを表すために使用する。
```PHP
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

@readonly ディレクティブは、指定した要素が"readonly"であるべきかを示すために使用する。
```PHP
<input type="email"
        name="email"
        value="email@laravel.com"
        @readonly($user->isNotAdmin()) />
```

@requiredディレクティブは、指定要素が"required"であることを表すために使用する。
```PHP
<input type="text"
        name="title"
        value="title"
        @required($user->isAdmin()) />
```

Bladeの@includeディレクティブを使用すると、別のビュー内からBladeビューを読み込める。
親ビューで使用できるすべての変数は、読み込むビューで使用できる。
```PHP
<div>
    @include('shared.errors')
    <form>
        <!-- フォームのコンテンツ -->
    </form>
</div>
```

読み込むビューは親ビューで使用可能なすべてのデータを継承するが、読み込むビューで使用可能にする必要がある追加データの配列を渡すこともできる。
```PHP
@include('view.name', ['status' => 'complete'])
```

存在しないビューを@includeしようとすると、Laravelはエラーを投げる。
存在する場合と存在しない場合があるビューを読み込む場合は、@includeIfディレクティブを使用する必要がある。
```PHP
@includeIf('view.name', ['status' => 'complete'])
```

指定する論理式がtrueかfalseと評価された場合に、ビューを@includeしたい場合は、@includeWhenおよび@includeUnlessディレクティブを使用できる。
```PHP
@includeWhen($boolean, 'view.name', ['status' => 'complete'])
@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

指定するビュー配列中、存在する最初のビューを含めるには、includeFirstディレクティブを使用する。
```PHP
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

ループと読み込みをBladeの@eachディレクティブで1行に組み合わせられる。
```PHP
@each('view.name', $jobs, 'job')
```
@eachディレクティブの最初の引数は、配列またはコレクション内の各要素に対してレンダするビュー。
2番目の引数は、反復する配列またはコレクションであり、3番目の引数は、ビュー内で現在の反復要素に割り当てる変数名。
現在の反復の配列キーは、ビュー内でkey変数として使用できるす。

@eachディレクティブに4番目の引数を渡すこともできる。
この引数は、指定された配列が空の場合にレンダするビューを指定する。
```PHP
@each('view.name', $jobs, 'job', 'view.empty')
```

@onceディレクティブを使用すると、レンダリングサイクルごとに1回だけ評価されるテンプレートの部分を定義できる。
```PHP
@once
    @push('scripts')
        <script>
            // カスタムJavaScript…
        </script>
    @endpush
@endonce
```

@onceディレクティブは、@pushや@prependディレクティブと一緒に使われることが多いため、便利なように@pushOnceと@prependOnceディレクティブがある。
```PHP
@pushOnce('scripts')
    <script>
        // カスタムJavaScript…
    </script>
@endPushOnce
```

Blade@phpディレクティブを使用して、テンプレート内でプレーンPHPのブロックを定義し、実行できる。
```PHP
@php
    $counter = 1;
@endphp
```

PHP文を1つ書くだけなら、@phpディレクティブ内に含められる。
```PHP
@php($counter = 1)
```

## コンポーネント
コンポーネントを作成するには、クラスベースのコンポーネントと匿名コンポーネントの2つのアプローチがある。

クラスベースのコンポーネントを作成するには、make:component Artisanコマンドを使用できる。
make:componentコマンドは、コンポーネントをapp/View/Componentsディレクトリに配置する。
```PHP
php artisan make:component Alert
```
make:componentコマンドは、コンポーネントのビューテンプレートも作成する。
ビューはresources/views/componentsディレクトリに配置される。
独自のアプリケーション用のコンポーネントを作成する場合、コンポーネントはapp/View/Componentsディレクトリとresources/views/componentsディレクトリ内で自動的に検出するため、通常それ以上のコンポーネント登録は必要ない。

サブディレクトリ内にコンポーネントを作成することもできる。
```PHP
php artisan make:component Forms/Input
```
上記のコマンドは、app/View/Components/FormsディレクトリにInputコンポーネントを作成し、ビューはresources/views/components/formsディレクトリに配置する。

匿名コンポーネント(Bladeテンプレートのみでクラスを持たないコンポーネント)を作成したい場合は、make:componentコマンドを実行するとき、--viewフラグを使用する。
```PHP
php artisan make:component forms.input --view
```
上記のコマンドは、resources/views/components/forms/input.blade.phpへBladeファイルを作成する。
このファイルは<x-forms.input />により、コンポーネントとしてレンダできる。

独自のアプリケーション用コンポーネントを作成する場合、コンポーネントをapp/View/Componentsディレクトリとresources/views/componentsディレクトリ内で自動的に検出する。

Bladeコンポーネントを利用するパッケージを構築する場合は、コンポーネントクラスとそのHTMLタグエイリアスを手作業で登録する必要がある。
通常、コンポーネントはパッケージのサービスプロバイダのbootメソッドに登録する必要がある。
```PHP
use Illuminate\Support\Facades\Blade;
/**
 * パッケージの全サービスの初期起動処理
 */
public function boot(): void
{
    Blade::component('package-alert', Alert::class);
}
```

コンポーネントを登録すると、タグエイリアスを使用してレンダリングできる。
```PHP
<x-package-alert/>
```

規約により、componentNamespaceメソッドを使用してコンポーネントクラスを自動ロードすることもできる。
たとえば、Nightshadeパッケージには、Package\Views\Components名前空間内にあるCalendarとColorPickerコンポーネントが含まれているとする。
```PHP
use Illuminate\Support\Facades\Blade;
/**
 * パッケージの全サービスの初期起動処理
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\Views\Components', 'nightshade');
}
```

package-name::構文を使用して、ベンダーの名前空間でパッケージコンポーネントが使用できるようになる。
```PHP
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Bladeは、コンポーネント名のパスカルケースを使い、コンポーネントにリンクしているクラスを自動的に検出する。
サブディレクトリもサポートしており、「ドット」表記を使用する。

コンポーネントを表示するために、Bladeテンプレート1つの中でBladeコンポーネントタグを使用できる。
Bladeコンポーネントタグは、文字列x-で始まり、その後にコンポーネントクラスのケバブケース名を続ける。
```PHP
<x-alert/>
<x-user-profile/>
```

コンポーネントクラスがapp/View/Componentsディレクトリ内のより深い場所にネストしている場合は、.文字を使用してディレクトリのネストを表せる。
たとえば、コンポーネントがapp/View/Components/Inputs/Button.phpにあるとしたら、以下のようにレンダリングする。
```PHP
<x-inputs.button/>
```

コンポーネントを条件付きでレンダしたい場合は、コンポーネントクラスでshouldRenderメソッドを定義する。
shouldRenderメソッドがfalseを返した場合、コンポーネントをレンダしない。
```PHP
use Illuminate\Support\Str;
/**
 * コンポーネントをレンダするか
 */
public function shouldRender(): bool
{
    return Str::length($this->message) > 0;
}
```

HTML属性を使用してBladeコンポーネントへデータを渡せる。
ハードコードするプリミティブ値は、単純なHTML属性文字列を使用してコンポーネントに渡すことができる。
PHPの式と変数は、接頭辞として:文字を使用する属性を介してコンポーネントに渡す必要がある。
```PHP
<x-alert type="error" :message="$message"/>
```

コンポーネントの全データ属性は、そのクラスコンストラクターで定義する必要がある。
コンポーネントのすべてのパブリックプロパティは、コンポーネントのビューで自動的に使用可能になる。
コンポーネントのrenderメソッドからビューにデータを渡す必要はない。
```PHP
namespace App\View\Components;
use Illuminate\View\Component;
use Illuminate\View\View;
class Alert extends Component
{
    /**
     * コンポーネントインスタンスを作成
     */
    public function __construct(
        public string $type,
        public string $message,
    ) {}
    /**
     * コンポーネントを表すビュー／コンテンツを取得
     */
    public function render(): View
    {
        return view('components.alert');
    }
}
```

コンポーネントをレンダするときに、変数を名前でエコーすることにより、コンポーネントのパブリック変数の内容を表示できる。
```PHP
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

コンポーネントコンストラクタの引数はキャメルケース（camelCase）を使用して指定する必要があるが、HTML属性で引数名を参照する場合はケバブケース（kebab-case）を使用する必要がある。
たとえば、次のようにコンポーネントコンストラクタに渡す。
```PHP
/**
 * コンポーネントインスタンスの作成
 */
public function __construct(
    public string $alertType,
) {}
```

$alertType引数は、次のようにコンポーネントに提供できる。
```PHP
<x-alert alert-type="danger" />
```PHP

コンポーネントに属性を渡す場合、「短縮」構文も使用できる。
```PHP
{{-- 短縮形 --}}
<x-profile :$userId :$name />
{{-- 同じ動作をする記法 --}}
<x-profile :user-id="$userId" :name="$name" />
```

JavaScriptフレームワークもコロンプレフィックスで属性を指定するため、属性がPHP式ではないことをBladeへ知らせるために二重のコロン(::)プレフィックスを使用する。
```PHP
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

コンポーネントの任意のパブリックメソッドを呼び出すこともできる。
```PHP
/**
 * 指定したオプションが現在選択されているオプションであるか判別
 */
public function isSelected(string $option): bool
{
    return $option === $this->selected;
}
```

メソッドの名前に一致する変数を呼び出すことにより、コンポーネントテンプレートでこのメソッドを実行できる。
```PHP
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

Bladeコンポーネントを使用すると、クラスのrenderメソッド内のコンポーネント名、属性、およびスロットにアクセスすることもできる。
このデータにアクセスするには、コンポーネントのrenderメソッドからクロージャを返す必要がある。
クロージャは、唯一の引数として$data配列を取ります。
この配列はコンポーネントに関する情報を提供するいくつかの要素が含んでいる。
```PHP
use Closure;
/**
 * コンポーネントを表すビュー／コンテンツを取得
 */
public function render(): Closure
{
    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];

        return '<div>Components content</div>';
    };
}
```
componentNameは、HTMLタグでx-プレフィックスの後に使用されている名前と同じ。
したがって、<x-alert/>のcomponentNameはalertになる。
attributes要素には、HTMLタグに存在していたすべての属性が含まれます。slot要素は、コンポーネントのスロットの内容を含むIlluminate\Support\HtmlStringインスタンス。

クロージャは文字列を返す必要がある。
返された文字列が既存のビューに対応している場合、そのビューがレンダリングされる。
それ以外の場合、返される文字列はインラインBladeビューとして評価される。

コンポーネントがLaravelのサービスコンテナからの依存を必要とする場合、コンポーネントのデータ属性の前にそれらをリストすると、コンテナによって自動的に注入される。
```PHP
use App\Services\AlertCreator;
/**
 * Create the component instance.
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {
```

パブリックメソッドまたはプロパティがコンポーネントテンプレートへ変数として公開されないようにする場合は、コンポーネントの$except配列プロパティへ追加する。
```PHP
namespace App\View\Components;
use Illuminate\View\Component;
class Alert extends Component
{
    /**
     * コンポーネントテンプレートに公開してはいけないプロパティ／メソッド。
     *
     * @var array
     */
    protected $except = ['type'];

    /**
     * コンポーネントインスタンスを作成
     */
    public function __construct(
        public string $type,
    ) {}
}
```

コンポーネントが機能するためには、不必要なclassなどの追加のHTML属性データを指定する必要が起きることもある。
通常これらの追加の属性は、コンポーネントテンプレートのルート要素に渡す。
たとえば、次のようにalertコンポーネントをレンダリングしたいとする。
```PHP
<x-alert type="error" :message="$message" class="mt-4"/>
```

コンポーネントのコンストラクタの一部ではないすべての属性は、コンポーネントの「属性バッグ」に自動的に追加される。
この属性バッグは、$attributes変数を通じてコンポーネントで自動的に使用可能になる。
この変数をエコーすることにより、すべての属性をコンポーネント内でレンダリングできる。
```PHP
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

属性のデフォルト値を指定したり、コンポーネントの属性の一部へ追加の値をマージしたりする必要のある場合は、属性バッグのmergeメソッドを使用できる。
このメソッドは、コンポーネントに常に適用する必要のあるデフォルトのCSSクラスのセットを定義する場合、特に便利。
```PHP
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

特定の条件がtrueである場合に、クラスをマージしたい場合classメソッドで実行でき、クラスの配列を引数に取る。
配列のキーには追加したいクラスを含み、値に論理式を指定する。
配列要素に数字キーがある場合は、レンダするクラスリストへ常に含まれる。
```PHP
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

他の属性をコンポーネントにマージする必要がある場合は、mergeメソッドをclassメソッドへチェーンできるs。
```PHP
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

class属性ではない属性をマージする場合、mergeメソッドへ指定する値は属性の「デフォルト」値と見なします。
class属性とは異なり、こうした属性は挿入した属性値とマージしない。
代わりに上書きする。
たとえば、buttonコンポーネントの実装は次のようになる。
```PHP
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

カスタムtypeを指定してボタンコンポーネントをレンダすると、このコンポーネントを使用するときにそれが指定される。
タイプが指定されない場合、buttonタイプを使用する。
```PHP
<x-button type="submit">
    Submit
</x-button>
```

class以外の属性にデフォルト値と挿入する値を結合させたい場合は、prependsメソッドを使用する。
この例では、data-controller属性は常にprofile-controllerで始まり、追加で挿入するdata-controller値はデフォルト値の後に配置される。
```PHP
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

filterメソッドを使用して属性をフィルタリングできる。
このメソッドは、属性を属性バッグへ保持する場合にtrueを返す必要があるクロージャを引数に取る。
```PHP
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

キーが特定の文字列で始まるすべての属性を取得するために、whereStartsWithメソッドを使用できる。
```PHP
{{ $attributes->whereStartsWith('wire:model') }}
```

whereDoesntStartWithメソッドは、指定する文字列で始まるすべての属性を除外するために使用する。
```PHP
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

firstメソッドを使用して、特定の属性バッグの最初の属性をレンダできる。
```PHP
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

属性がコンポーネントに存在するか確認する場合は、hasメソッドを使用できる。
このメソッドは、属性名をその唯一の引数に取り、属性が存在していることを示す論理値を返す。
```PHP
@if ($attributes->has('class'))
    <div>Class attribute is present</div>
@endif
```

配列をhasメソッドへ渡した場合、このメソッドは与えた属性がすべてコンポーネントに存在するかを判定する。
```PHP
@if ($attributes->has(['name', 'class']))
    <div>All of the attributes are present</div>
@endif
```

hasAnyメソッドは、指定属性のいずれかがコンポーネントに存在するかを判定するために使用する。
```PHP
@if ($attributes->hasAny(['href', ':href', 'v-bind:href']))
    <div>One of the attributes is present</div>
@endif
```

getメソッドを使用して特定の属性の値を取得できる。
```PHP
{{ $attributes->get('class') }}
```

多くの場合、「スロット」を利用して追加のコンテンツをコンポーネントに渡す必要がある。
コンポーネントスロットは、$slot変数をエコーしレンダする。
```PHP
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

コンポーネントにコンテンツを挿入することにより、そのコンテンツをslotに渡せる。
```PHP
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

コンポーネント内の異なる場所へ、複数の異なるスロットをレンダする必要のある場合がある。
「タイトル（"title"）」スロットへ挿入できるようにするため、アラートコンポーネントを変更してみる。
```PHP
<span class="alert-title">{{ $title }}</span>
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

x-slotタグを使用して、名前付きスロットのコンテンツを定義できる。
明示的にx-slotタグ内にないコンテンツは、$slot変数のコンポーネントに渡される。
```PHP
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

「スコープ付きスロット」は、スロットの中でコンポーネントのデータやメソッドへアクセスできる機構。
コンポーネントにパブリックメソッドまたはプロパティを定義し、$component変数を使いスロット内のコンポーネントにアクセスすることで、Laravelと同様の動作を実現できる。
この例では、x-alertコンポーネントのコンポーネントクラスにパブリックformatAlertメソッドが定義されていると想定している。
```PHP
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

ブレードコンポーネントと同様に、CSSクラス名などのスロットに追加の属性を割り当てることができるs。
```PHP
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>
    Content
    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

スロット属性を操作するため、スロットの変数のattributesプロパティへアクセスできる。
```PHP
@props([
    'heading',
    'footer',
])
<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>
    {{ $slot }}
    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

コンポーネントのマークアップをrenderメソッドから直接返すことができる。
```PHP
/**
 * コンポーネントを表すビュー／コンテンツを取得
 */
public function render(): string
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

インラインビューをレンダするコンポーネントを作成するには、make:componentコマンドを実行するときにinlineオプションを使用する。
```PHP
php artisan make:component Alert --inline
```

あるコンポーネントをレンダする必要があっても、実行時までどのコンポーネントをレンダすべきか分からない場合、Laravelの組み込みコンポーネントであるdynamic-componentを使用すると、実行時の値や変数に基づいてコンポーネントをレンダできる。
```PHP
// $componentName = "secondary-button";
<x-dynamic-component :component="$componentName" class="mt-4" />
```

独自のアプリケーション用コンポーネントを作成する場合、コンポーネントをapp/View/Componentsディレクトリとresources/views/componentsディレクトリ内で自動的に検出する。

Bladeコンポーネントを利用するパッケージを構築する場合や、コンポーネントをデフォルト外のディレクトリに配置する場合は、コンポーネントクラスとそのHTMLタグエイリアスを手作業で登録し、Laravelがコンポーネントを探す場所を認識できるようにする必要がある。
通常、パッケージのサービスプロバイダのbootメソッドでコンポーネントを登録する必要がある。
```PHP
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;
/**
 * パッケージの全サービスの初期起動処理
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

コンポーネントを登録すると、タグエイリアスを使用してレンダリングできる。
```PHP
<x-package-alert/>
```

規約により、componentNamespaceメソッドを使用してコンポーネントクラスを自動ロードすることもできる。
たとえば、Nightshadeパッケージには、Package\Views\Components名前空間内にあるCalendarとColorPickerコンポーネントが含まれているとする。
```PHP
use Illuminate\Support\Facades\Blade;
/**
 * パッケージの全サービスの初期起動処理
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\Views\Components', 'nightshade');
}
```
これにより、package-name::構文を使用して、ベンダーの名前空間でパッケージコンポーネントが使用できるようになる。
```PHP
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Bladeは、コンポーネント名のパスカルケースを使い、コンポーネントにリンクしているクラスを自動的に検出する。
サブディレクトリもサポートしており、「ドット」表記を使用する。

## 匿名コンポーネント
インラインコンポーネントと同様に、匿名コンポーネントは、単一のファイルを介してコンポーネントを管理するためのメカニズムを提供します。
匿名コンポーネントは単一のビューファイルを利用し、関連するクラスはない。
匿名コンポーネントを定義するには、resources/views/componentsディレクトリ内にBladeテンプレートを配置するだけで済む。
たとえば、resources/views/components/alert.blade.phpでコンポーネントを定義すると、以下のようにレンダできる。
```PHP
<x-alert/>
```

.文字を使用して、コンポーネントがcomponentsディレクトリ内のより深い場所にネストしていることを示せる。
たとえば、コンポーネントがresources/views/components/input/button.blade.phpで定義されていると仮定すると、次のようにレンダリングできる。
```PHP
<x-inputs.button/>
```

あるコンポーネントが多数のBladeテンプレートから構成されている場合、そのコンポーネントのテンプレートを1つのディレクトリにまとめたいことがある。
```PHP
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

このディレクトリ構造を使用すると、アコーディオン・コンポーネントとそのアイテムを以下のようにレンダできる。
```PHP
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

x-accordionでアコーディオンコンポーネントをレンダするには、他のアコーディオン関連のテンプレートと一緒にaccordionディレクトリ内に入れ子にするのではなく、"index"アコーディオン・コンポーネントテンプレートをresources/views/componentsディレクトリ内に配置する必要がある。


Bladeでは幸い、コンポーネントのテンプレートディレクトリ内に index.blade.php ファイルを配置することができる。
index.blade.php`のテンプレートがコンポーネントに存在する場合、そのテンプレートはコンポーネントの「ルート」ノードとしてレンダされる。
そこで、上記の例で示したのと同じBladeの構文を引き続き使用することができるが、ディレクトリ構造を次のように調整する。
```PHP
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

匿名コンポーネントには関連付けたクラスがないため、どのデータを変数としてコンポーネントに渡す必要があり、どの属性をコンポーネントの属性バッグに配置する必要があるか、どう区別するか疑問に思われるかもしれない。

コンポーネントのBladeテンプレートの上部にある@propsディレクティブを使用して、データ変数と見なすべき属性を指定できる。
コンポーネント上の他のすべての属性は、コンポーネントの属性バッグを介して利用する。
データ変数にデフォルト値を指定する場合は、変数の名前を配列キーに指定し、デフォルト値を配列値として指定する。
```PHP
<!-- /resources/views/components/alert.blade.php -->
@props(['type' => 'info', 'message'])
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

上記のコンポーネント定義をすると、そのコンポーネントは次のようにレンダできる。
```PHP
<x-alert type="error" :message="$message" class="mb-4"/>
```

子コンポーネントの中にある親コンポーネントのデータにアクセスしたい場合は、@awareディレクティブを使用する。
例えば、親の<x-menu>と子の<x-menu.item>で構成される複雑なメニューコンポーネントを作っている。
```PHP
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

<x-menu>コンポーネントは、以下のような実装になる。
```PHP
<!-- /resources/views/components/menu/index.blade.php -->
@props(['color' => 'gray'])
<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

colorプロップは親(<x-menu>)にしか渡されていないため、<x-menu.item>の中では利用できない。
@awareディレクティブを使用すれば、<x-menu.item>内でも利用可能になる。
```PHP
<!-- /resources/views/components/menu/item.blade.php -->
@aware(['color' => 'gray'])
<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

匿名コンポーネントは通常、resources/views/componentsディレクトリに、Bladeテンプレートを配置することにより定義する。

anonymousComponentPathメソッドは、最初の引数に匿名コンポーネントの場所の「パス」、第2番引数としてコンポーネントを配置するための「名前空間」をオプションで取る。
通常、このメソッドはアプリケーションのサービスプロバイダのbootメソッドから呼び出す必要がある。
```PHP
/**
 * アプリケーションの全サービスの初期起動処理
 */
public function boot(): void
{
    Blade::anonymousComponentPath(__DIR__.'/../components');
}
```

上記の例のように、コンポーネントパスへプレフィックスを指定せず登録した場合、対応するプレフィックスを指定していないBladeコンポーネントと同様にレンダーする。
例えば、上記で登録したパスにpanel.blade.phpコンポーネントが存在する場合、以下のようにレンダーされる。
```PHP
<x-panel />
```

プレフィックスの「名前空間」は、anonymousComponentPathメソッドの第2引数として与える。
```PHP
Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');
```

プレフィックスを指定すると、その「名前空間」内のコンポーネントは、コンポーネントがレンダーされるとき、そのコンポーネントの名前空間にプレフィックスを付けレンダーされる。
```PHP
<x-dashboard::panel />
```

## レイアウト構築
layoutコンポーネントの例。
```PHP
<!-- resources/views/components/layout.blade.php -->
<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

layoutコンポーネントを定義したら、コンポーネントを利用するBladeビューを作成できる。
```PHP
<!-- resources/views/tasks.blade.php -->
<x-layout>
    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

コンポーネントへ挿入されるコンテンツは、layoutコンポーネント内のデフォルト$slot変数として提供される。
layoutも提供されるのであれば、$titleスロットを尊重する。
それ以外の場合は、デフォルトのタイトルが表示される。
```PHP
<!-- resources/views/tasks.blade.php -->
<x-layout>
    <x-slot:title>
        カスタムタイトル
    </x-slot>

    @foreach ($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

レイアウトとタスクリストのビューを定義したので、ルートからtaskビューを返す必要がある。
```PHP
use App\Models\Task;
Route::get('/tasks', function () {
    return view('tasks', ['tasks' => Task::all()]);
});
```

##  テンプレート継承を使用するレイアウト
レイアウトは、「テンプレート継承」を介して作成することもできる。
```PHP
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```
名前が暗示するように、@sectionディレクティブは、コンテンツのセクションを定義するが、@yieldディレクティブは指定するセクションの内容を表示するために使用する。

子ビューを定義するときは、@extends Bladeディレクティブを使用して、子ビューが「継承する」のレイアウトを指定する。
Bladeレイアウトを拡張するビューは、@sectionディレクティブを使用してレイアウトのセクションにコンテンツを挿入できる。
上記の例で見たように、これらのセクションの内容は@yieldを使用してレイアウトへ表示できる。
```PHP
<!-- resources/views/child.blade.php -->

@extends('layouts.app')
@section('title', 'Page Title')
@section('sidebar')
    @@parent
    <p>これはマスターサイドバーに追加される</p>
@endsection
@section('content')
    <p>これは本文の内容</p>
@endsection
```
この例では、sidebarのセクションは、@parentディレクティブを利用して、(上書きするのではなく)、レイアウトのサイドバーにコンテンツを追加している。
@parentディレクティブは、ビューをレンダするときにレイアウトの内容へ置き換えられる。

@yieldディレクティブは、デフォルト値を2番目の引数に取る。
この値は、生成するセクションが未定義の場合にレンダされる。
```PHP
@yield('content', 'Default content')
```


## フォーム
アプリケーションでHTMLフォームを定義したときは、CSRF保護ミドルウェアがリクエストをバリデートできるように、フォームへ隠しCSRFトークンフィールドを含める必要がある。
@csrf Bladeディレクティブを使用してトークンフィールドを生成できる。
```PHP
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```PHP

HTMLフォームはput、patch、またはdeleteリクエストを作ることができないので、これらのHTTP動詞を偽装するために_Method隠しフィールドを追加する必要がある。
@methodBladeディレクティブは、皆さんのためこのフィールドを作成する。
```PHP
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

指定する属性にバリデーションエラーメッセージが存在するかをすばやく確認するために@errorディレクティブを使用できる。
@errorディレクティブ内では、エラーメッセージを表示するため、$message変数をエコーできる。
```PHP
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

@errorディレクティブは"if"文へコンパイルされるため、属性のエラーがない場合にコンテンツをレンダしたい場合は、@elseディレクティブを使用できる。
```PHP
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror">
```

ページが複数のフォームを含んでいる場合にエラーメッセージを取得するため、特定のエラーバッグの名前を第2引数へ渡せる。
```PHP
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror">

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## スタック
Bladeを使用すると、別のビューまたはレイアウトのどこか別の場所でレンダできる名前付きスタックに入れ込む。
これは、子ビューに必要なJavaScriptライブラリを指定する場合、とくに便利。
```PHP
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

もし、指定した論理式が、trueと評価された場合にコンテンツを@pushしたいのであれば、@pushIfディレクティブを使用する。
```PHP
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

必要な回数だけスタックに入れ込める。
スタックしたコンテンツを完全にレンダするには、スタックの名前を@stackディレクティブに渡す。
```PHP
<head>
    <!-- HEADのコンテンツ -->

    @stack('scripts')
</head>
```

スタックの先頭にコンテンツを追加する場合は、@prependディレクティブを使用する。
```PHP
@push('scripts')
    これは２番めになる
@endpush

// Later...

@prepend('scripts')
    これが最初になる
@endprepend
```


## サービス注入
@injectディレクティブは、Laravelサービスコンテナからサービスを取得するために使用する。
@injectに渡す最初の引数は、サービスが配置される変数の名前であり、2番目の引数は解決したいサービスのクラスかインターフェイス名。
```PHP
@inject('metrics', 'App\Services\MetricsService')
<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

## インラインBladeテンプレートのレンダ
素のBladeテンプレート文字列を有効なHTMLに変換する必要がある場合、Bladeファサードが提供するrenderメソッドを使用する。
renderメソッドはBladeテンプレート文字列と、テンプレートに提供するデータの配列をオプションで受け取る。
```PHP
use Illuminate\Support\Facades\Blade;
return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

LaravelはインラインのBladeテンプレートをstorage/framework/viewsディレクトリに書き込むことによってレンダする。
Bladeテンプレートをレンダリングした後、Laravelにこれらの一時ファイルを削除させたい場合は、このメソッドにdeleteCachedView引数を指定する。
```PHP
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```


## Bladeフラグメントのレンダ
Turboやhtmxなどのフロントエンドフレームワークを使用する場合は、HTTPレスポンスの中からBladeテンプレートの一部のみを返す必要が起きる場合があり、Bladeの「フラグメント(fragment)」は、それを可能にする。
まず、Bladeテンプレートの一部を@fragmentディレクティブと@endfragmentディレクティブの中に配置する。
```PHP
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

その後、このテンプレートを利用するビューをレンダする際に、fragmentメソッドを呼び出し、指定したフラグメントのみを送信HTTPレスポンスへ含めるように指示する。
```PHP
return view('dashboard', ['users' => $users])->fragment('user-list');
```

fragmentIfメソッドを使用すると、指定条件に基づき、ビューの断片を条件付きで返せる。
そうでなければ、ビュー全体を返す。
```PHP
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

fragmentsとfragmentsIfメソッドを使用すると、レスポンスへ複数のビューフラグメントを返せる。
フラグメントは一つに連結する。
```PHP
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

## Bladeの拡張
Bladeでは、directiveメソッドを使用して独自のカスタムディレクティブを定義できる。
Bladeコンパイラは、カスタムディレクティブを検出すると、ディレクティブに含まれる式を使用して、提供したコールバックを呼び出す。

次の例は、指定した$varをフォーマットする@datetime($var)ディレクティブを作成する。
これはDateTimeインスタンスである必要がある。
```PHP
namespace App\Providers;
use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;
class AppServiceProvider extends ServiceProvider
{
    /**
     * アプリケーションの全サービスの登録
     */
    public function register(): void
    {
        // ...
    }
    /**
     * 全アプリケーションサービスの初期起動処理
     */
    public function boot(): void
    {
        Blade::directive('datetime', function (string $expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

ご覧のとおり、ディレクティブに渡される式にformatメソッドをチェーンする。
したがって、この例でディレクティブが生成する最終的なPHPは、以下のよ​​うになる。
```PHP
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

Bladeを使ってオブジェクトを「エコー」しようとすると、そのオブジェクトの__toStringメソッドが呼び出される。
操作するクラスがサードパーティのライブラリへ属している場合など、特定のクラスで__toStringメソッドを制御できない場合が起こりえる。

このような場合、Bladeで、その特定のタイプのオブジェクト用に、カスタムEchoハンドラを登録できる。
Bladeのstringableメソッドを呼び出す。
stringableメソッドは、クロージャを引数に取る。
このクロージャは、自分がレンダリングを担当するオブジェクトのタイプをタイプヒントで指定する必要がある。
一般的には、アプリケーションのAppServiceProviderクラスのbootメソッド内で、stringableメソッドを呼び出す。
```PHP
use Illuminate\Support\Facades\Blade;
use Money\Money;
/**
 * アプリケーションの全サービスの初期起動処理
 */
public function boot(): void
{
    Blade::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

カスタムEchoハンドラを定義したら、Bladeテンプレート内のオブジェクトを簡単にエコーできる。
```PHP
Cost: {{ $money }}
```

カスタムディレクティブのプログラミングは、単純なカスタム条件ステートメントを定義するとき、必要以上の複雑さになる場合がある。
そのため、BladeにはBlade::ifメソッドが用意されており、クロージャを使用してカスタム条件付きディレクティブをすばやく定義できる。
たとえば、アプリケーションのデフォルト「ディスク」の設定をチェックするカスタム条件を定義する。
これは、AppServiceProviderのbootメソッドで行う。
```PHP
use Illuminate\Support\Facades\Blade;
/**
 * アプリケーションの全サービスの初期起動処理
 */
public function boot(): void
{
    Blade::if('disk', function (string $value) {
        return config('filesystems.default') === $value;
    });
}
```

カスタム条件を定義すると、テンプレート内で使用できる。
```PHP
@disk('local')
    <!-- アプリケーションはローカルディスクを使用している -->
@elsedisk('s3')
    <!-- アプリケーションはs3ディスクを使用している -->
@else
    <!-- アプリケーションは他のディスクを使用している -->
@enddisk

@unlessdisk('local')
    <!-- アプリケーションはローカルディスクを使用していない -->
@enddisk
```