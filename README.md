# CustomCarCount
> 自定义控件，类似于京东购物车上添加或者减少商品数量的功能.还可以手动输入数量，以及数量限制

[代码对应博客](http://blog.csdn.net/u014702332/article/details/53611377)



####实现的功能
1. 实现按钮添加与减少数量
2. 手动输入数量
3. 解决输入超出最小值与最大值范围，重新赋值造成卡死问题


首先我写了一个监听数量变化的接口
```

/**
 * 监听选择技能数量的变化
 */
public interface IChangeCoutCallback {

    void change(int count);

}

```

#####使用按钮控制加减操作

1. 控制按钮可用与不可用 
2. 设置当前编辑框中的数据，以及光标
3. 执行回调，告诉使用页，数据的变化

```

    /**
     * 添加操
     */
    private void addAction() {
        countValue++;
        btnChangeWord();
    }
    
    /**
     * 删除操作
     */
    private void minuAction() {
        countValue--;
        btnChangeWord();
    }

    private void btnChangeWord() {
        ivMinu.setEnabled(countValue > MIN_VALUE);
        ivAdd.setEnabled(countValue < maxValue);
        etCount.setText(String.valueOf(countValue));
        etCount.setSelection(etCount.getText().toString().trim().length());
        callback.change(countValue);
    }

```

####实现输入改变数据
1. 这里的数据如果小于最小值，或者超过最大值的时候，我们重新赋值，把它设为最小值或者最大值
2. 这里重新赋值后，会出现循环赋值的bug,造成卡死，所以我添加了一个needUpdate判断，
3. 如果输入空的时候，则不重新对当前编辑框赋值，只对外面数据更改，所以需要判断是否为空


```


    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {

    }

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {

    }

    @Override
    public void afterTextChanged(Editable s) {
        boolean needUpdate = false;
        if (!TextUtils.isEmpty(s)) {
            countValue = Integer.valueOf(s.toString());
            if (countValue <= MIN_VALUE) {
                countValue = MIN_VALUE;
                ivMinu.setEnabled(false);
                ivAdd.setEnabled(true);
                needUpdate = true;
                Toast.makeText(mContext, String.format("最少添加%s个数量", MIN_VALUE), Toast.LENGTH_SHORT).show();
            } else if (countValue >= maxValue) {
                countValue = maxValue;
                ivMinu.setEnabled(true);
                ivAdd.setEnabled(false);
                needUpdate = true;
                Toast.makeText(mContext, String.format("最多只能添加%s个数量", maxValue), Toast.LENGTH_SHORT).show();

            } else {
                ivMinu.setEnabled(true);
                ivAdd.setEnabled(true);
            }
        } else {  //如果编辑框被清空了，直接填1
            countValue = 1;
            ivMinu.setEnabled(false);
            ivAdd.setEnabled(true);
            needUpdate = true;
            Toast.makeText(mContext, String.format("最少添加%s个数量", MIN_VALUE), Toast.LENGTH_SHORT).show();

        }
        changeWord(needUpdate);
    }


     /***needupdate 是否需要移除监听**/
    private void changeWord(boolean needUpdate) {
        if (needUpdate) {
            etCount.removeTextChangedListener(this);
            if (!TextUtils.isEmpty(etCount.getText().toString().trim())) {  //不为空的时候才需要赋值
                etCount.setText(String.valueOf(countValue));
            }
            etCount.addTextChangedListener(this);
        }
        etCount.setSelection(etCount.getText().toString().trim().length());
        callback.change(countValue);
    }
```








###如何使用这个自定义控件
1.在布局里面引入
```
        <com.king_mi.customcarcount.CounterView
            android:id="@+id/cv_counter"
            android:layout_width="wrap_content"
            android:layout_marginRight="10dp"
            android:layout_height="wrap_content" />

```

2.当前类中使用
```

public class MainActivity extends AppCompatActivity {

    @BindView(R.id.cv_counter)
    CounterView cvCounter;
    @BindView(R.id.tv_show_count)
    TextView tvShowCount;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        cvCounter.setMaxValue(5);
        cvCounter.setCallback(callback);
    }


    private IChangeCoutCallback callback = new IChangeCoutCallback() {
        @Override
        public void change(int count) {            //总价变化
            tvShowCount.setText("我变成了" + count);
        }
    };
}


```
