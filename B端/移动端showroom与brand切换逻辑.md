##移动端showroom与brand切换逻辑
###先看上面这些

define kMaxReRequestFor402 5 402的重复请求最多执行5次，超过之后返回到登陆页面（登出操作）

####网络请求基类方法中处理
	* 如果requestUrl为kGetShowroomBrandToken或者kShowroomToBrand。则会将responseHeaders中的token写入kTempUserLoginTokenKey
	* 如果requestUrl为kShowroomBrandToShowroom或者其他。则会将responseHeaders中的token写入kUserLoginTokenKey

#####相关接口描述：
	kShowroomToBrand  /service/showroom/showroomToBrand（从showroom获取brand信息接口）
	
	kGetShowroomBrandToken  /service/showroom/getShowroomBrandToken （brand页面中token失效处理）
	
	kShowroomBrandToShowroom  /service/showroom/brandToShowroom  （返回showroom接口，brand（showroom———>brand）需要重新登陆时调用）

####相关方法：
	/**
	 * 判断当前用户角色是否是brand角色（showroom———>brand）
	 * 判断本地的kTempUserLoginTokenKey对应的有没有值
	 * 有值返回true
	 * 没有值返回false
	 */
#####+(BOOL)isShowroomToBrand;

	/**
	 * 获取当前用户token
	 * 通过isShowroomToBrand方法，判断是否是brand角色（showroom———>brand）
	 * 如果不是、通过kUserLoginTokenKey获取token
	 * 如果是、通过kTempUserLoginTokenKey获取token
	 */
#####+(NSString *)getToken；

	/**
	 * 清空 kTempUserLoginTokenKey、kTempBrandID
	 * 其实就是使brand角色（showroom———>brand）登出
	 * 使用场景
	 * 1、     app启用的时候
	 * 2、    网络请求基类返回kCode402（需要重新登录），并且isShowroomToBrand返回ture时调用，并且402超过kMaxReRequestFor402（5次）；
	 * 3、    从brand主页返回到showroom主页的时候调用
	 * 3.1、 有网络的情况下，kShowroomBrandToShowroom请求成功后才会调用。
	 * 3.2、无网络的情况下，会调用
	 * 4、  showroom主页，扫码流程未完成时（即在用户角色成功切换到品牌之后，未进入到款式详情页）
	 */
#####+ (void )removeTempUser

	/**
	 * 获取BrandID
	 * 通过isShowroomToBrand方法，判断是否是brand角色（showroom———>brand）
	 * 如果是、通过kTempBrandID获取BrandID，并返回
	 * 如果不是、通过kYYUserBrandIDKey获取BrandID，并返回
	 */
#####+(NSString *)getBrandID

####关于登陆操作：
	1、在有网络的情况下，每次重新打开app都会登陆一次（kill之后打开）
	2、如果usertype 是5或者6的时候，会跳转到showroom页面。如果不是跳转到主页。

###再看下面的

####showroom到brand逻辑  
	1、点击主页showroom主页cell，请求kShowroomToBrand接口。
	2、请求成功后，将获取到对应brand用户信息。
	3、并将brandID（brandId）保存到本地（kTempBrandID）。
	4、跳转到brand主页。

####brand到showroom逻辑
	1、在brand主页，点击返回showroom按钮。请求kShowroomBrandToShowroom接口。
	2、请求成功后，将获取到showroom信息，并将之前存的brand用户的临时信息清空（kTempUserLoginTokenKey，kTempBrandID）。
	3、清空购物车,最后返回到showroom主页。


####所以

#####切换到brand（showroom——>brand）状态条件
	点击主页showroom主页cell，进入到brand主页

#####切换到showroom （或者说brand角色登出） 条件 + (void )removeTempUser
	 * 1、     app启动的时候（在kill的情况下重新打开）
	 * 2、    网络请求基类返回kCode402（需要重新登录），并且isShowroomToBrand返回ture时调用，并且402超过kMaxReRequestFor402（5次）；
	 * 3、    从brand主页返回到showroom主页的时候调用
	 * 3.1、 有网络的情况下，kShowroomBrandToShowroom请求成功后才会调用。
	 * 3.2、无网络的情况下，会调用

