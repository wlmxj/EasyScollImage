# EasyScollImage
仿小红书登录页 长图片背景滚动效果

 - 先看小红书的效果

![小红书.GIF](https://upload-images.jianshu.io/upload_images/5739496-42991c168b713160.GIF?imageMogr2/auto-orient/strip)

- ######说说思路

滚动效果用RecyclerView实现。RecyclerView有个smoothScrollToPosition方法，可以滚动到指定位置（有滚动效果，不是直接到指定位置），不了解的看这里[RecycleView4种定位滚动方式演示](https://www.jianshu.com/p/3acc395ae933)。每一个Item是一张长图，这样首尾相接滚动起来（滚到无限远）就是无限循环的效果，然后再改变滚动的速度，完成。

```
public class MainActivity extends AppCompatActivity {

    private RecyclerView mRecyclerView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        //全屏
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);

        setContentView(R.layout.activity_main);

        mRecyclerView = findViewById(R.id.mRecyclerView);
        mRecyclerView.setAdapter(new SplashAdapter(MainActivity.this));
        mRecyclerView.setLayoutManager(new ScollLinearLayoutManager(MainActivity.this));

        //smoothScrollToPosition滚动到某个位置（有滚动效果）
        mRecyclerView.smoothScrollToPosition(Integer.MAX_VALUE / 2);
    }


}
```

- ######无限循环
将RecyclerView的Item数量设置成很大的值，用smoothScrollToPosition方法滚到很远的位置，就能实现这样的效果，很多banner轮播图的实现也是如此；
```
public class SplashAdapter extends RecyclerView.Adapter<SplashAdapter.ViewHolder> {

    private int imgWidth;


    public SplashAdapter(Context context) {
        imgWidth = EasyUtil.getScreenWidth(context);
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_splash, parent, false);
        return new ViewHolder(itemView);
    }


    @Override
    public void onBindViewHolder(final ViewHolder holder, final int position) {
       /* ViewGroup.LayoutParams lp = holder.item_bg.getLayoutParams();
        lp.width = imgWidth;
        lp.height =imgWidth*5;
        holder.item_bg.setLayoutParams(lp);*/
    }

    @Override
    public int getItemCount() {
        return Integer.MAX_VALUE;
    }

    public class ViewHolder extends RecyclerView.ViewHolder {

        ImageView item_bg;

        public ViewHolder(final View itemView) {
            super(itemView);
            item_bg = itemView.findViewById(R.id.item_bg);
        }

    }

}
```

- ######控制smoothScrollToPosition的滑动速度

参考[RecyclerView调用smoothScrollToPosition() 控制滑动速度](https://blog.csdn.net/a86261566/article/details/50906456)，修改MILLISECONDS_PER_INCH的值即可
```

/**
 * 更改RecyclerView滚动的速度
 */
public class ScollLinearLayoutManager extends LinearLayoutManager {
    private float MILLISECONDS_PER_INCH = 25f;  //修改可以改变数据,越大速度越慢
    private Context contxt;

    public ScollLinearLayoutManager(Context context) {
        super(context);
        this.contxt = context;
    }


    @Override
    public void smoothScrollToPosition(RecyclerView recyclerView, RecyclerView.State state, int position) {
        LinearSmoothScroller linearSmoothScroller =
                new LinearSmoothScroller(recyclerView.getContext()) {
                    @Override
                    public PointF computeScrollVectorForPosition(int targetPosition) {
                        return ScollLinearLayoutManager.this
                                .computeScrollVectorForPosition(targetPosition);
                    }

                    @Override
                    protected float calculateSpeedPerPixel
                            (DisplayMetrics displayMetrics) {
                        return MILLISECONDS_PER_INCH / displayMetrics.density;
                        //返回滑动一个pixel需要多少毫秒
                    }

                };
        linearSmoothScroller.setTargetPosition(position);
        startSmoothScroll(linearSmoothScroller);
    }

    //可以用来设置速度
    public void setSpeedSlow(float x) {
        //自己在这里用density去乘，希望不同分辨率设备上滑动速度相同
        //0.3f是自己估摸的一个值，可以根据不同需求自己修改
        MILLISECONDS_PER_INCH = contxt.getResources().getDisplayMetrics().density * 0.3f + (x);
    }

}
```

- ######图片宽度充满屏幕、高度按图片原始宽高比例自适应
```
@SuppressLint("AppCompatCustomView")
public class FitImageView extends ImageView {

    public FitImageView(Context context) {
        super(context);
    }

    public FitImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
        Drawable drawable = getDrawable();

        if(drawable!=null){
            int width = MeasureSpec.getSize(widthMeasureSpec);
            int height = (int) Math.ceil((float) width * (float) drawable.getIntrinsicHeight() / (float) drawable.getIntrinsicWidth());
            setMeasuredDimension(width, height);
        }else{
            super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        }
    }

}
```
**这里需要注意的是、Item的根布局  android:layout_height="wrap_content"，否则图片高度会受限**

```
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <com.next.scrollimagedemo.view.FitImageView
        android:id="@+id/item_bg"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:src="@mipmap/ww1" />


    <!--  <ImageView
          android:id="@+id/item_bg"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:src="@mipmap/ww2"
          android:scaleType="centerCrop"/>-->

</android.support.constraint.ConstraintLayout>
```

- ######使RecyclerView不能手指触碰滑动
加层View屏蔽掉事件就好了
```
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/mRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></android.support.v7.widget.RecyclerView>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#80000000"
        android:clickable="true" />

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="80dp"
        android:layout_marginTop="80dp"
        app:layout_constraintTop_toTopOf="parent"
        android:scaleType="centerInside"
        android:src="@mipmap/slogan"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent" />

</android.support.constraint.ConstraintLayout>
```
- ######完成效果

![WW.GIF](https://upload-images.jianshu.io/upload_images/5739496-aefd9b8eb4c59e5a.GIF?imageMogr2/auto-orient/strip)

---
#Demo
github：[https://github.com/forvv231/EasyScollImage](https://github.com/forvv231/EasyScollImage)

apk：[https://fir.im/gfdj]( https://fir.im/gfdj)

![image.png](https://upload-images.jianshu.io/upload_images/5739496-748feb5b85af3a23.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

#By the way

不要吐槽这背景图片、不喜欢、Demo代码里面还有别的(￣▽￣)~*

#点个赞吧
年初给自己定了今年存3万块钱的目标，刚才自己算了一下，还差8万…
