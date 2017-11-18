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
### 五、控制器接收视图的数据
* 全部数据
```php
 $input=Input::all();
 ```
 * 接收除了某些字段的部分数据
 ```php
 $input=Input::except('_token','_method');
 ```
 * 分页显示
 (1)控制器部分
 ```php
 $data=Article::orderby('article_id','desc')->paginate(5);//根据article_id降序查询，每页显示5条数据
 return view('admin/article/index',compact('data'));//把数据传到视图
 ```
 (2)视图部分
 ```php
 {{$data->links()}}
 ```
 ### 七、session的使用
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
 ### 六、添加数据
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
### 七、更新数据
(1)选择要修改的记录
```php
$field=Article::find($article_id);//查找对应的记录
return view('edit',compact('field'));//把查到的记录数据传到视图
```
(2)执行更新
```php
$input=Input::except('_token','_method');//接收除了_token、_method字段外的数据
$res=Article::where('article_id',$article_id)->update($input);//找到对应的article_id执行修改
if($res){
    return redirect('admin/article/index');//如果修改成功返回主页
}else{
    return back()->with('errors','更新失败');//否则提示更新失败
}
```
### 八、删除某条记录
```php
$res=Article::where('article_id',$article_id)->delete();//找到对应的article_id执行删除
if($res){
    return redirect('admin/article/index');//如果删除成功返回主页
}else{
    return back()->with('errors','更新失败');//否则提示删除失败
}
```
