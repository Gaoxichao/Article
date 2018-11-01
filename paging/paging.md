���¹�˾����࣬���һֱ�ڲ��� Andorid  �Ŷӵļܹ����������������ͼƬѡ�����ʹ���� paging ��Ϊ��ҳ���ؿ�ܡ�˳���Ķ���һ��paging��Դ�롣�������¼һ�¡�

���νӳ� paging�� ���ܻ�һ���±ƣ��о������˺ܶ� API�� ��֪�����������֡������ȶ� paging ����ɲ��ֽ���һ���˽⡣

���ȣ����ǰ��� `�б��ҳ����` �����Ϊ����һ�������Ļ��֣���Ϊ 2 �����֣� `����` �� `UI`�� paging ���ǰ�����������л��ֵ�

###### ����

���ݲ��� paging ����
* `PagedList` һ���̳��� `AbstractList` �� `List` ���࣬ ����������Դ��ȡ������
* `DataSource` ����Դ�ĸ���ֱ��ṩ�� [PageKeyedDataSource](https://developer.android.google.cn/reference/android/arch/paging/PageKeyedDataSource)��[ItemKeyedDataSource](https://developer.android.google.cn/reference/android/arch/paging/ItemKeyedDataSource)��[PositionalDataSource](https://developer.android.google.cn/reference/android/arch/paging/PositionalDataSource)�� ������Դ�У����ǿ��Զ��������Լ������ݼ����߼���


###### UI
UI ���� paging �ṩ��һ���µ� `PagedListAdapter`, ��ʵ������� `Adapter` ��ʱ��������Ҫ�ṩһ���Լ�ʵ�ֵ� `DiffUtil.ItemCallback` ���� `AsyncDifferConfig`


##### ����

�Է�ҳ����Դ `PageKeyedDataSource` Ϊ��

����һ������Դ�� ���� Language Ϊ demo �е�ʵ�����

```kotlin
class LanguageDataSource: PageKeyedDataSource<Int, Language>()
```

ʵ������ override ����

```kotlin
override fun loadInitial(params: LoadInitialParams<Int>, callback: LoadInitialCallback<Int, Language>) {
}
```

```koltin
override fun loadAfter(params: LoadParams<Int>, callback: LoadCallback<Int, Language>) {
}
```

```kotlin
override fun loadBefore(params: LoadParams<Int>, callback: LoadCallback<Int, Language>) {
}
```

�� 3 �����������ν���Ϊ

* ���μ���
* ����һҳ����
* ǰһҳ����


���Ǹ���һҳ��������߼�

```kotlin
LanguageRepository.requestLanguages({datas->
    if (datas.code == 200) {
        val languages = datas.data
        Handler(Looper.getMainLooper()).post {
           callback.onResult(languages, null, 1)
        }
    } else {
    }
 }, {t->
     Log.e(javaClass.simpleName, "${t.message}")
})
```

���� `LanguageRepository` ������ `retrofit` ������һ�� Language ������б� ���ǵ���
`callback.onResult` �ͻ�ˢ�� RecyclerView ����ͼ

`loadAfter` ��ʵ�ִ����� `loadInitial` һ�£����ﲻ��׸����

����������һ�� UI �㣬���Ƕ���һ�� `PagedListAdapter`

```kotlin
class LanguageAdapter(private val context: Context) : PagedListAdapter<Language, ViewHolder>(languageDiff)
```

����������Ҫ override 2������

```kotlin
 override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder
```

```kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int)
```

�� `onBindViewHolder` �У� ���ǿ���ͨ�� `getItem(position)` ��ȡ����ڵ�����ʵ��ȥ���� UI ��չʾ��


��������һ���ȽϹؼ��Ĳ��֣��Ǿ���������� DATA �� UI �������֡�

```kotlin
val config = PagedList.Config.Builder()
    .setPageSize(15)
    .setPrefetchDistance(2)
    .setInitialLoadSizeHint(15)
    .setEnablePlaceholders(false)
    .build()

val pageList = PagedList.Builder(LanguageDataSource(), config)
    .setNotifyExecutor {
         Handler(Looper.getMainLooper()).post {it.run()}
    }
    .setFetchExecutor(Executors.newFixedThreadPool(2))
    .build()

 adapter.submitList(pageList)
```

����� pageList �� `NotifyExecutor` ��  `FetchExecutor` Ҳ�Ǳ������õġ��� `Android arch componet` �����ļܹ��У����Ƽ�ʹ�ù���һ�� `PageList` �� `LiveData` �ķ�ʽ�����ǲ�ʹ��Ҳû�й�ϵ��`arch compoent` ���������������ﲻ��������������������ϸʹ�ÿ��Բ鿴[google��ʵ��Դ��](https://github.com/googlesamples/android-architecture-components/tree/master/PagingWithNetworkSample)


�ڴ����˽��� paging ����ɲ��ֺ����ǻῪʼ���棬�����ǵ���Ϊʲô��Ҫ paging �أ� ��������֮ǰ��ͨ��ʹ�÷�ʽ��ʲô�����أ����ǿ�����Դ����Ѱ�ҵ��𰸡�

���ǿ����� 2 �����ֵ������ԽӴ���Ϊ�������з������鿴 `PagedList.Builder#build()` ��Դ�룺

```java
return PagedList.create(
    mDataSource,
    mNotifyExecutor,
    mFetchExecutor,
    mBoundaryCallback,
    mConfig,
    mInitialKey);
```

�����鿴

```java
return new ContiguousPagedList<>(contigDataSource,
    notifyExecutor,
    fetchExecutor,
    boundaryCallback,
    config,
    key,
    lastLoad);
```

���������Ĺ��췽�������Կ��������߼�
```java
mDataSource.dispatchLoadInitial(key,
    mConfig.initialLoadSizeHint,
    mConfig.pageSize,
    mConfig.enablePlaceholders,
    mMainThreadExecutor,
    mReceiver);
```

������ `PageKeyedDataSource` Ϊ���� ������ `DataSource` ����ͬ��

�鿴 `dispatchLoadInital` ����

```java
LoadInitialCallbackImpl<Key, Value> callback =
                new LoadInitialCallbackImpl<>(this, enablePlaceholders, receiver);
loadInitial(new LoadInitialParams<Key>(initialLoadSize, enablePlaceholders), callback);

callback.mCallbackHelper.setPostExecutor(mainThreadExecutor);
```

�������ǿ��Կ����� `loadInitial` ����������Ҫ�� override �ķ���֮һ��������������� callback �� onResult �������׷�����ʲô�أ�

�鿴 `LoadInitialCallbackImpl#onResult()` ��Դ�룬�ؼ��߼�����

```java
mDataSource.initKeys(previousPageKey, nextPageKey);
int trailingUnloadedCount = totalCount - position - data.size();
if (mCountingEnabled) {
    mCallbackHelper.dispatchResultToReceiver(new PageResult<>(
    data, position, trailingUnloadedCount, 0));
} else {
    mCallbackHelper.dispatchResultToReceiver(new PageResult<>(data, position));
}
```

�鿴 `dispatchResultToReceiver`

![image](http://upload-images.jianshu.io/upload_images/1523772-4c2c96a6a49a05b3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

�����鿴 `onPageResult` ����

![image](http://upload-images.jianshu.io/upload_images/1523772-fa18ac3648e50d15?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

���ǹ�עһ�� `init` ʱ����߼�

```java
mStorage.init(pageResult.leadingNulls, page, pageResult.trailingNulls,
                        pageResult.positionOffset, ContiguousPagedList.this);
```

`init` ���߼��ܼ򵥣�ֻ�� 2 ��

```java
init(leadingNulls, page, trailingNulls, positionOffset);
callback.onInitialized(size());
```

![image](http://upload-images.jianshu.io/upload_images/1523772-0260a6e1015f5ef3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

����� ���ǿ��Կ����ؼ����߼�

```java
mPages.clear();
mPages.add(page);
```

����� `PageList` �󶨵����ݾͷ����˱仯��֮�����ǰ� `PageList` submit ���� `adapter` ��ô�����ݾͷ����˸��¡�

��ʼ�������ǿ����ˣ���ô��ʣ�µ���������μ��ص���


���Ƿ������� `RecyclerView`�� ������ǻ����б��������������ʱ�򣬺���Ȼ����� adapter �� bind ��������ô������ȥ�鿴 `PagedListAdapter#getItem` ��Դ�롣

```java
return mDiffer.getItem(position);
```

![image](http://upload-images.jianshu.io/upload_images/1523772-2fe02912d9f45e88?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

�鿴 `PageList` �� `loadAround`

```java
loadAroundInternal(index);
```

������

![image](http://upload-images.jianshu.io/upload_images/1523772-4e6c31deedf40e54?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
if (mAppendItemsRequested > 0) {
    scheduleAppend();
}
```

�鿴 `scheduleAppend` ��ʵ��

```java
mBackgroundThreadExecutor.execute(new Runnable() {
            @Override
            public void run() {
                if (isDetached()) {
                    return;
                }
                if (mDataSource.isInvalid()) {
                    detach();
                } else {
                    mDataSource.dispatchLoadAfter(position, item, mConfig.pageSize,
                            mMainThreadExecutor, mReceiver);
                }
            }
        });
```

������ǿ����� `dispatchLoadAfter` �����ĵ��ã�֮����߼���֮ǰ�� `dispathLoadInitial` �ͷǳ��������ˡ�


���գ�����õ������߼�
![image](http://upload-images.jianshu.io/upload_images/1523772-b8f65ff6a14148b8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

������� `AsyncPagedListDiffer` �� `PagedList.Callback` �Ļص�

![image](http://upload-images.jianshu.io/upload_images/1523772-2142830b94143dab?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


���callback �Ǻ� adapter ���������ġ����Ի�������ˢ���б�

������ǿ�һ�� `Adapter` �� submit �����������Կ����������߼�
![image](http://upload-images.jianshu.io/upload_images/1523772-acc7aacfc4bb6b78?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

���ǿ��Կ��� paging �������� `DiffUtils` �� RecyclerView ����ˢ�µġ���������Ҳ���赣�� paging ������������⡣


##### ���

���̸һ�¶� paging ����⡣ һ������£�������ԭʼ�ķ�ʽ���б� UI ���ڵĲ��֣�����Ҫ֪�����ݵ���Դ���߼����֣������ڳ����� mvp ģʽ�У�������ݺ� UI ���зֲ㡣 �� paging ������һϵ�еķ�װ�� �ṩ�˸���ͨ�õ� API ����������Щ���顣��ͨ�׵�˵������ʵ���˷�ҳ���ؽṹ�е� Presenter �㼰 Presenter������δ����֡�

����ģʽ��ҵ��ı�д�ߣ����԰� UI ���ֵĴ���ģ�廯�� ֻ��Ҫ����ҵ���߼������Ұ�ҵ���߼��е����ݻ�ȡд�� DataSource �У�ʹ��ҳ���صĲ�������̶ȸ��ߡ�