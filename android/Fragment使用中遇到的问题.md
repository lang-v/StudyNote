# Fragment使用中遇到的问题

## 带有参数的构造方法导致Crash

在Fragment.java 的注释上强调不建议重写构造方法，一定要有一个无参的构造方法，原因是在Fragment恢复状态时会使用反射调用无参构造方法。

例如在横竖屏切换时，就会发生Fragment的状态恢复，就会导致找不到无参构造方法，出现Fragment$InstantiationException

```java
/**
 * Create a new instance of a Fragment with the given class name.  This is
 * the same as calling its empty constructor, setting the {@link ClassLoader} on the
 * supplied arguments, then calling {@link #setArguments(Bundle)}.
 *
 * @param context The calling context being used to instantiate the fragment.
 * This is currently just used to get its ClassLoader.
 * @param fname The class name of the fragment to instantiate.
 * @param args Bundle of arguments to supply to the fragment, which it
 * can retrieve with {@link #getArguments()}.  May be null.
 * @return Returns a new fragment instance.
 * @throws InstantiationException If there is a failure in instantiating
 * the given fragment class.  This is a runtime exception; it is not
 * normally expected to happen.
 * @deprecated Use {@link FragmentManager#getFragmentFactory()} and
 * {@link FragmentFactory#instantiate(ClassLoader, String)}, manually calling
 * {@link #setArguments(Bundle)} on the returned Fragment.
 */
@Deprecated
@NonNull
public static Fragment instantiate(@NonNull Context context, @NonNull String fname,
        @Nullable Bundle args) {
    try {
        Class<? extends Fragment> clazz = FragmentFactory.loadFragmentClass(
                context.getClassLoader(), fname);
        Fragment f = clazz.getConstructor().newInstance();
        if (args != null) {
            args.setClassLoader(f.getClass().getClassLoader());
            f.setArguments(args);
        }
        return f;
    .....
```



解决办法：

1. 设置当前活动，不处理屏幕旋转事件
2. 去除构造方法中的参数，使用fragment.setArgments()传递参数；使用fragment.getArgments()；获取参数



## 