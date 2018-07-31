# java.lang.IllegalArgumentException No view found for id 0x7

布局上找不到这个View，看下ERROR的地方，可以看到是因为Container找不到因此报错了。

从布局入手，参考了很多文章主要分为以下两种原因

## 1 Fragment 嵌套了 Fragment 
 
例如：FragmentA 里面 嵌套了一个FragmentB

如果在FragmentA中用FragmentB 的时候使用了getFragmentManager() 那么就会抛这个异常，正确的做法是使用getChildFragmentManager()。

因为，getFragmentManager() 获取的是Activity的布局，所以FragmentB 是找不到Activity中的Container的，应该去FragmentA中找Container，因此要使用getChildFragmentManager()，这个获取的就是Fragment的fragmentManger

## 2 使用DataBindingUtils错误设置布局导致的

注意在Fragment 中使用DataBindingUtils 需要用inflate，不要用成setContentView了
```
@Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        binding = DataBindingUtil.inflate(inflater,getLayoutId(),null,false);
        viewModel = onCreateViewModel();
        binding.setVariable(BR.viewModel, viewModel);
        viewModel.onAttach(this);
        return binding.getRoot();
    }
```