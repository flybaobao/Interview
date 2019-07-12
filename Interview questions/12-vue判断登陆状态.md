问题：

 (vue.js)前后端分离的单页应用如何来判断当前用户的登录状态？

描述:

有个单页应用的url例如`http://cctv.com/!#/car/list`,只有在登录的情况下，我才可以去访问这个url，如果不是登录状态，则要跳到登录页面。

以前的话，请求url到后台，后台会判断下当前用户是否登录，但是现在做成单页应用了，也不需要请求到后台了，那么在单页应用的情况下如何来处理用户是否已经登录了的状态呢？

------

解决方案1:

具体看你后台怎么支持，token和cookie这俩种方法都可以

解决方案2:

无论什么情况，前端都不知道用户是谁，用户是否登录。那谁知道呢？后端！

所以，你每次只要把后端给你的登录态信息再给后端，让后端校验就行了。

1、你可以存在cookie中，每次请求发出去。当然，这个对你可以是透明的，让后端来种cookie，同时设置成http only的。

2、因为是单页，也可以存在内存的某个全局变量中，如果没有则未登录。

3、可以自己缓存token，每次发请求手动带过去。

解决方案3:

让后台set cookie就行了。

后台set cookie后，前端在请求的时候，header都会自动带过去的。

然后约定一个状态码，当请求回来的是特定的状态码的时候，就跳转到登录页面。

解决方案4:

登陆状态由后端存入cookie就可以了。你不用考虑。

解决方案5:

通过localStorange存储登录状态，不过毫无安全性可言

解决方案6:

现在通常的做法是前端把用户名和密码通过api发送到后台，至于是否保持登录状态，由后台去设置cookie，前台不需要做任何事情

解决方案7:

登陆后后端api给token，前端存储token，访问需要登录的传token过去。前端路由中判断，是否有token，无就转登录。另外前端有异步交互api的错误处理，比如后端错误代码是token过期了什么的错误，前端清除之前token，转登录。

解决方案8:



```
history.listen(location => {
  const pathname = location.pathname;
  if (pathname !== '/login' && pathname !== '/logout') {
    dispatch({
      type: 'check',
      payload: pathname
    });
  }else if(pathname === '/logout'){
    dispatch({
      type: 'logout',
      payload: pathname
    });
  }
});
*check({ payload }, { select, call, put }) {
  const { isLogin, lastCheck, interval } = yield select( ({ auth }) => auth);
  const now = (new Date()).getTime();
  yield put({
    type: 'authRedirect',
    payload,
  })
  if (!isLogin){
    yield put(routerRedux.push('/login'));
    return ;
  }
  //是否异步检测
  if (!interval || (now - lastCheck) < interval  ){
    return;
  }
  const { data, err } = yield call(fresh);
  if (data && data.status==0) {
    yield put({
      type: 'freshSuccess',
      payload: data.data,
    });
    return ;
  }
  yield put(routerRedux.push('/login'));
}
// fresh login status
export async function fresh(params) {
  return request(`/api/auth/fresh?${qs.stringify(params)}`);
}
```

监听 `url` 变化，如果不是login logout 就检查登录状态。
检查登录状态

1. 检查js变量，没有登录就直接跳登录页
2. 如果js变量已经登录，就判断一下是否需要异步检测 不需要检测就结束(比如上次检测是在60秒内)。
3. 如果需要异步检测，就异步检测是否登录，如果成功 刷新一下`lastcheck`时间。
4. 如果检测没有登录，就直接跳登录页