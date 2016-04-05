# ViewHolder 模式

Pan原生实现了ViewHolder模式，开发者无需自行绑定ViewHolder。

## ListView，Adapter和ViewModel的嵌套

在ListView或GridView等可重复布局中，都需要使用到Adapter，一般来说，通过继承BaseAdapter并实现相应方法来完成。

使用ListView的界面中，界面实际上被分为了两层：

1. 列表层，即ListView，以及与ListView同级的其他View，例如list.xml
2. Item层，即ListView中的每一个元素，同时，可能还有headerView和footerView以及其他自定义的View，同处这一层，例如item.xml

这两层基本没有关系，只通过Adapter建立了微弱的联系，Item其实并不需要知道自己身处在哪个ListView中，而ListView也不太关心底层的Item的具体实现。因此，ViewModel也可以同样使用两层，以明确这种弱关系，同时解耦两层的渲染逻辑。

首先是ListViewModel，主要负责ListView层面的渲染，同时在Adapter中使用ItemViewModel把Item的渲染代理出去。

```Java
@Xml(R.layout.list)
public class ListViewModel extends GeneralViewModel{

    @Bind(R.id.list)
    public ListView vListView;

    private List<Data> mDatas;

    private BaseAdapter mAdapter;

    @Override protected void onInit(){
    	vListView.setAdapter(mAdapter = new ListViewAdapter());
    }
	
	@Override protected void render(){
		mAdapter.notifyDatasetChanged();
	}

	public ListViewModel setData(List<Data> datas){
		mDatas = datas;
		return this;
	}

	class ListViewAdapter extends BaseAdapter{
		//...

        @Override
		public View getView(int position, View convertView, ViewGroup parent){

			//两层ViewModel只有非常弱的联系
			return Pan.with(getActivity(), ItemViewModel.class)
			    .getViewModel(parent, convertView, false).
			    .setData(datas.get(position))
			    .getRootView();
		}
	}
}
```

其次是ItemViewModel，ItemViewModel并不需要知道自己被用在了何处，可以是Activity中某个部分使用了，也可能是Adapter中使用了，但ItemViewModel并不关心。

```Java
@Xml(R.layout.item)
public class ItemViewModel extends GeneralViewModel{

    @Bind(R.id.text)
    public TextView vTextView;

    private Data mData;

    public ItemViewModel setData(Data data){
    	mData = data;
    	return this;
    }

    @Override protected void render(){
    	vTextView.setText(mData.toString());
    }
	
}
```

最后，在Activity中的调用，也很简洁，只需要绑定ListViewModel。

```Java

//在Activity中
protected void onCreate(Bundle savedInstance){
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_main);

	Pan.with(this, ListViewModel.class)
	    .controlledBy(ListViewController.class)
		.getViewModel();
}

```

## RecyclerViewModel

RecyclerViewModel用于RecyclerView中，由于Java的单继承，RecyclerView.Adapter需要返回一个ViewHolder的子类对象，因此，这里使用RecyclerViewModel作为基类以替换GeneralViewModel。其他用法和ListView完全一致。

```Java
@Xml(R.layout.item)
public class RecyclerItemViewModel extends RecyclerViewModel{
	//...
}

//Adapter
public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerItemViewModel>{
	
	@Override RecyclerItemViewModel onCreateViewHoler(ViewGroup parent, int viewType){
		return Pan.with(getLifecycleObserved(), RecyclerItemViewModel.class)
				.getViewModel(parent, null, false);
	}

	@Override void onBindViewHolder(RecyclerItemViewModel holder, int position){
		//holder.setData(...)
	}
}
```


