---
title: Java自定义注解
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

**通过注解动态 findViewById**

定义一个 BindView 注解

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface BindView {
    int value();
}
```

示例代码

```java
public class MainActivity extends AppCompatActivity {
    @BindView(R.id.tv)
    private TextView tv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
        initValue();
    }

    private void initView() {
        Class clazz = MainActivity.class;
        for (Field field : clazz.getDeclaredFields()) {
            if (field.isAnnotationPresent(BindView.class)) {
                BindView annotation = field.getAnnotation(BindView.class);
                int viewId = annotation.value();
                try {
                    field.set(MainActivity.this, findViewById(viewId));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void initValue() {
        tv.setText("Hello World");
    }
}
```

参考

+ [Java自定义注解入门到实战](https://juejin.cn/post/6982471491568812040)