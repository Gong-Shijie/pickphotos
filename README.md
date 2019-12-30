# pickphotos

![危险权限申请流程](https://upload-images.jianshu.io/upload_images/19741117-c466aeab59b0aaa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
调用相册过程就涉及了对于危险权限读取存储的申请、隐式跳转到相册   
一个极简的demo方便回顾这两个点：
简书：https://www.jianshu.com/p/08ef5e2da5e0
#### 权限
由于Android对于应用需要的权限做了分类处理分为了危险权限和普通权限，对于普通权限的申请可以直接在AndroidManifest里面申请即可  
但是对于一些涉及用户隐私和财产的权限被列为危险权限，如果需要使用这些危险权限，就必须在程序使用的时候进行授权，运行时权限申请。  

#####  危险权限  
![危险权限表](https://upload-images.jianshu.io/upload_images/19741117-fb783a28425435f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



因此对于开发时候需要什么样的权限，如果是普通权限直接申请，如果是危险权限则走流程进行运行时权限。  
**判断是否授权和请求授权**
```
//判断是否授权这里以一个权限为例
if (ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
//没有授权进行权限申请
      ActivityCompat.requestPermissions(MainActivity.this, new String[]{ Manifest.permission. WRITE_EXTERNAL_STORAGE }, 1);      
```
**用户授权结束活动后回调**  
```
 public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    openAlbum();
                } else {
                    Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }
```
**隐式意图跳转**
```
 private void openAlbum() {
        Intent intent = new Intent("android.intent.action.GET_CONTENT");
        intent.setType("image/*");
        startActivityForResult(intent, CHOOSE_PHOTO); // 打开相册
    }
```
**活动结束后的回调得到图片的Uri**
```
  @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode){
            case CHOOSE_PHOTO:
                if(resultCode == RESULT_OK){
                   //使用Glide来加载图片data.getData()得到图片的Uri
                    Glide.with(this).load(data.getData()).into(photo);
                }
        }
    }
```
***
####  隐式启动intent
隐式启动intent来跳转到相应的活动，根据action和category同时匹配要跳转的意图  
这两个属性可以在AndroidManife.xml里面配置每个活动用来响应的action和category  
一个Activity只有一个action但是可以有多个category  
但是需要考虑intent并不智能比如启动相册：  
需要采用隐式启动   
 ```
Intent intent = new Intent("android.intent.action.GET_CONTENT");
        intent.setType("image/*");
        startActivityForResult(intent, CHOOSE_PHOTO); // 打开相册
```                            
意图指定action为获取相应的内容， 但是不知道要获取什么内容，只有通过setType指定为image这样这个意图才可以成功跳转到相册。    
这里由于是采用了startActivityForResult函数启动意图，在跳转到相册执行了操作后，得到的数据就在onActivityResult里面处理返回的数据。                                                  
***

####  常见的action：

ACTION_MAIN                               作为一个主要的进入口，而并不期望去接受数据
ACTION_VIEW                              向用户去显示数据
ACTION_ATTACH_DATA      用于指定一些数据应该附属于一些其他的地方，例如，图片数据应该附属于联系人
ACTION_EDIT                                访问已给的数据，提供明确的可编辑
ACTION_PICK                                从数据中选择一个子项目，并返回你所选中的项目
ACTION_CHOOSER                        显示一个activity选择器，允许用户在进程之前选择他们想要的
ACTION_GET_CONTENT                 允许用户选择特殊种类的数据，并返回（特殊种类的数据：照一张相片或录一段音）
ACTION_DIAL                                拨打一个指定的号码，显示一个带有号码的用户界面，允许用户去启动呼叫
ACTION_CALL                                根据指定的数据执行一次呼叫
（ACTION_CALL在应用中启动一次呼叫有缺陷，多数应用ACTION_DIAL，ACTION_CALL不能用在紧急呼叫上，紧急呼叫可以用ACTION_DIAL来实现） 
ACTION_SEND                               传递数据，被传送的数据没有指定，接收的action请求用户发数据
ACTION_SENDTO                           发送一跳信息到指定的某人
ACTION_ANSWER                          处理一个打进电话呼叫
ACTION_INSERT                            插入一条空项目到已给的容器
ACTION_DELETE                           从容器中删除已给的数据
ACTION_RUN                                运行数据，无论怎么
ACTION_SYNC                              同步执行一个数据
ACTION_PICK_ACTIVITY               为已知的Intent选择一个Activity，返回别选中的类
ACTION_SEARCH                         执行一次搜索
ACTION_WEB_SEARCH                执行一次web搜索
***
####  category：
CATEGORY_APP_BROWSER 和ACTION_MAIN一起使用，用来启动浏览器应用程序
CATEGORY_APP_CALCULATOR 和ACTION_MAIN一起使用，用来启动计算器应用程序
CATEGORY_APP_CALENDAR 和ACTION_MAIN一起使用，用来启动日历应用程序
CATEGORY_APP_CONTACTS 和ACTION_MAIN一起使用，用来启动联系人应用程序
CATEGORY_APP_EMAIL 和ACTION_MAIN一起使用，用来启动邮件应用程序
CATEGORY_APP_GALLERY 和ACTION_MAIN一起使用，用来启动图库应用程序
CATEGORY_APP_MAPS 和ACTION_MAIN一起使用，用来启动地图应用程序
CATEGORY_APP_MARKET 这个activity允许用户浏览和下载新的应用程序
CATEGORY_APP_MESSAGING ACTION_MAIN一起使用，用来启动短信应用程序
CATEGORY_APP_MUSIC 和ACTION_MAIN一起使用，用来启动音乐应用程序
CATEGORY_BROWSABLE  能够被浏览器安全调用的activity必须支持这个   
***
####  主要代码以打开相册选择图片为例子：
#####  权限组问题   
虽然有权限组的存在但是申请一个组内的权限应该遵循将需要的权限都列出的习惯。  
```
ActivityCompat.requestPermissions(MainActivity.this, new String[]
{
Manifest.permission.WRITE_EXTERNAL_STORAGE,
Manifest.permission.READ_EXTERNAL_STORAGE},
 GET_STORAGE_PERMISSION);
}
```
#####  通过返回的Uri显示图片  
因为Android KitKat后对于Uri做了封装所以对于回调中的数据Uri的取出得到图片路径的逻辑有些复杂，我们不重复制造轮子取出图片的path用bitmap加载图片  
######  使用Glide框架来加载图片  
######  添加依赖
> implementation 'com.github.bumptech.glide:glide:4.10.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.10.0'

简单加载图片方法：
```
 //使用Glide来加载图片data.getData()得到图片的Uri
                    Glide.with(this).load(data.getData()).into(photo);             
```
#####  MainActivity代码：
```
public class MainActivity extends AppCompatActivity {

    private static final int CHOOSE_PHOTO = 1;
    private static final int GET_STORAGE_PERMISSION = 2;
    private Button pick;
    private ImageView photo;
    private String imagepath = "";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        pick = findViewById(R.id.pick_btn);
        photo = findViewById(R.id.photo);


        pick.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                    //没有授权进行权限申请
                    ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE,Manifest.permission.READ_EXTERNAL_STORAGE}, GET_STORAGE_PERMISSION);}
                else{
                    pickPhoto();
                }
            }
        });
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case GET_STORAGE_PERMISSION:
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    pickPhoto();
                } else {
                    Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show();
                }
                break;

        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode){
            case CHOOSE_PHOTO:
                if(resultCode == RESULT_OK){
                   //使用Glide来加载图片data.getData()得到图片的Uri
                    Glide.with(this).load(data.getData()).into(photo);
                }
        }
    }

    private void pickPhoto() {
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("image/*");
        startActivityForResult(intent, CHOOSE_PHOTO); // 打开相册
    }
}
```
***
#####  总结：申请权限调用对应的应用核心流程在于对于几个回调函数的处理

>参考：书籍：<<第一行代码>>