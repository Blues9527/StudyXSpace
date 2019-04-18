
# ViewModel

    ViewModel是一个抽象类，里面只有一个方法。

```
protected 非抽象方法
onCleared() void;
```

用法：

Activity:
```
public class UserActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.user_activity_layout);
        final UserModel viewModel = ViewModelProviders.of(this).get(UserModel.class);
        viewModel.userLiveData.observer(this, new Observer<User>() {
            @Override
            public void onChanged(@Nullable User data) {
                // update ui.
            }
        });
        findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                 viewModel.doAction();
            }
        });
    }
}
```

ViewModel:
```
public class UserModel extends ViewModel {
      public final LiveData<User> userLiveData = new LiveData<>();
 
      public UserModel() {
          // trigger user load.
     }
 
      void doAction() {
         // depending on the action, do necessary business logic calls and update the
          // userLiveData.
      }
  }
```

Fragment:

```
public class MyFragment extends Fragment {
    public void onStart() {
        UserModel userModel = ViewModelProviders.of(getActivity()).get(UserModel.class);
    }
}
```