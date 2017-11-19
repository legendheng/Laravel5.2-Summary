# Laravel5.2-Summary
Laravel5.2的基础部分总结
### 一、路由的创建
```php
有三种方法包括get、post、any，其中any包括这get和post
Route::any('login','Admin\LoginController@login');
Route::get('code','Admin\LoginController@code');
```
### 二、视图的渲染
```php
return view('admin.login')->with('admin',$adminname);;//admin目录下的login页面，并带参数$adminname的值
```
### 如果视图提交时post的话，需要加上以下代码
```php
{{csrf_field()}}
```
### 三、返回原来的页面
```php
return back()->with('message','用户名或者密码错误');//返回原来的登陆页面，并且带上错误信息
```
```php
@if(session('message'))
    <p class="tip">{{session('message')}}</p>  //视图接收返回的错误信息
@endif
```
### 四、使用asset方法引入js、css
```php
<script type="text/javascript" src="{{asset('resources/views/admin/static/lib/jquery/1.9.1/jquery.min.js')}}"></script>
```
### 五、路由中间件的使用(限制活动开始为例)
(1)新建一条中间件路由
```php
在Kernel.php的 protected $routeMiddleware = [];加上下面这句
'activity' => \App\Http\Middleware\Activity::class,
```
(2)新建两条活动路由，一条是未开始的，一条是开始的
```
Route::any('start',['uses'=>'ActivityController@start']);//活动开始
Route::group(['middleware'=>['activity']],function(){
    Route::any('unsatrt',['uses'=>'ActivityController@unstart']);//活动未开始
});
```
(3)新建中间件文件写验证规则
```php
在Middleware目录下新建Activity.php文件，并写上以下
use Closure;
class Activity{
    public function handle($request, Closure $next){
        if(time()<strtotime('2017-11-11')){
            return redirect('unstart');//如果时间未到11月11日就跳会unstart页面提示为开始
        }
        return $next($request);
    }
}
```
(3)控制器的start和unstart方法
```php
  public function unstart(){
       return '活动尚未开始，~请稍等~';
    }

    public function start(){
        return '活动正在进行';
    }
```
(4)直接访问start路由，如果在时间大于11月11日就可以进入活动开始页面，否则进入活动为开始页面
### 六、控制器接收视图的数据
* 接收全部数据
```php
 $input=Input::all();
 ```
 * 接收除了某些字段的部分数据
 ```php
 $input=Input::except('_token','_method');
 ```
 * 分页显示
 (1)控制器部分(orm方法)
 ```php
 $data=Article::orderby('article_id','desc')->paginate(5);//根据article_id降序查询，每页显示5条数据
 return view('admin/article/index',compact('data'));//把数据传到视图
 ```
 (2)视图部分
 ```php
 {{$data->links()}}
 ```
 ### 七、session的使用(自带是120s)，在config目录下sesion.php可以修改
 ```php
 (1)首先要在根目录的index.php开启session服务,session_start();
 (2)然后控制器采可以存放session值，
 session(['user'=>$user['name']]);
 $request->session->put(key,value);//这是使用Request方法
 (3)判断是否存在，
 if ($request->session()->has('users')) {}
 (4)其次控制器的获取session，
 public function showProfile(Request $request, $id)
    {
        $value = $request->session()->get('key');//获取key的session值
        $data = $request->session()->all();//获取全部session
    }
 (5)视图获取session，
 {{ Session::get('nickname') }}
 ```
 ### 八、添加数据create方法(orm模式),并结合validator验证数据
 `注意：要先设定好模型的fillable属性(支持操作)或者guarded属性(不支持操作)，这样的目的是为了防止用户随意修改某些属性`
 ```php
 protected $fillable=['name','age'];//允许操作的字段
 protected $guarded=[]; //不允许操作的字段 
 ```
 * 首先要新建一个model，此例为article
 ```php
 使用atrisan创建model
 php artisan make:model Article
 ```
 * 其次控制器接收数据，使用validator验证数据
 ```php
 (1)$input=Input::except('_token');//接收除了_token字段之外的数据
 (2)$rules=[
            'art_content'=>'required', //设置验证规则
           ];
    $message=[
        'art_content.required'=>'文章内容不能为空', / /设置错误是返回的信息 
             ];
    $validator=\Illuminate\Support\Facades\Validator::make($input,$rules,$message);
    if($validator->passes()){
        $res=Article::create($input);      //使用create方法创建  
        if($res){
            return redirect('admin/article');
        }else{
            return back()->with('errors','错误');
        }
    }else{
//      dd($validator->errors()->all());打印所有错误
        return back()->withErrors($validator);  //视图接收错误信息
    }
 ```
 (3)视图接收错误信息
 ```php
 @if(count($errors)>0)
    @if(is_object($errors))
        @foreach($errors->all() as $error)
            <p style="font-size: 20px;color: red">{{$error}}</p>
        @endforeach
    @else
        <p style="font-size: 20px;color: red">{{$errors}}</p>
    @endif
@endif
```
### 九、删除某条记录delete方法(orm模式)
```php
$res=Article::where('article_id',$article_id)->delete();//找到对应的article_id执行删除
if($res){
    return redirect('admin/article/index');//如果删除成功返回主页
}else{
    return back()->with('errors','更新失败');//否则提示删除失败
}
```
### 十、更新数据save方法(orm模式)
(1)选择要修改的记录
```php
$field=Article::find($article_id);//查找对应的记录
return view('edit',compact('field'));//把查到的记录数据传到视图
```
(2)执行更新
```php
$input=Input::except('_token','_method');//接收除了_token、_method字段外的数据
$res=Article::where('article_id',$article_id)->save($input);//找到对应的article_id执行修改
if($res){
    return redirect('admin/article/index');//如果修改成功返回主页
}else{
    return back()->with('errors','更新失败');//否则提示更新失败
}
```
 ### 十一、查询数据(orm模式)
 `注意一：model模型的名字的复数会默认是查询表。如模型名称user会默认是users表，可以按下面修改`
 ```php
 protected $table = 'admin_user';//这是操作表
 protected $primarykey = 'id';//设置主键
 ```
 `注意二：默认情况下表是需要有update_at、created_at这两个字段的，用于自动存时间戳，不需要时可以按下面修改`
 ```php
 protected $timestamps = fales;
 ```
 * 查询所有模型数据
 ```php
 $data=Article::all();
 ```
 * 条件查询
 ```php
 $data=Article::where('view'.'>',100)->get();
 ```
 * 拆分查询(可以减少内存一下子的消耗量)
 ```php
 Article::chunk(500,function($student){
     var_dump($student);  //每次取出500条记录
 }) ;
 * 查询comment字段存在值的记录
 ```php
 $data=Article::has('comment')->get();
 ```
 * 模糊查询
 ```php
 $data=Article::whereHas('comment',function($q){
  $q->where('content','like','_me%');
 })
 ```
 ### 十二、DB查询构造器
 (1)$data = DB::table('user')->get();
 (2)$data = DB::table('user')->where('name', 'John')->get();
 (3)$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
