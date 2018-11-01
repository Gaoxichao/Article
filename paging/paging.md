���¹�˾����࣬���һֱ�ڲ��� Andorid �Ŷӵļܹ����������������ͼƬѡ�����ʹ���� paging ��Ϊ��ҳ���ؿ�ܡ�˳���Ķ���һ��paging��Դ�롣�������¼һ�¡�

���νӳ� paging�� ���ܻ�һ���±ƣ��о������˺ܶ� API�� ��֪�����������֡������ȶ� paging ����ɲ��ֽ���һ���˽⡣

���ȣ����ǰ��� �б��ҳ���� �����Ϊ����һ�������Ļ��֣���Ϊ 2 �����֣� ���� �� UI�� paging ���ǰ�����������л��ֵ�

����
���ݲ��� paging ����

PagedList һ���̳��� AbstractList �� List ���࣬ ����������Դ��ȡ������
DataSource ����Դ�ĸ���ֱ��ṩ�� PageKeyedDataSource��ItemKeyedDataSource��PositionalDataSource�� ������Դ�У����ǿ��Զ��������Լ������ݼ����߼���
UI
UI ���� paging �ṩ��һ���µ� PagedListAdapter, ��ʵ������� Adapter ��ʱ��������Ҫ�ṩһ���Լ�ʵ�ֵ� DiffUtil.ItemCallback ���� AsyncDifferConfig

����
�Է�ҳ����Դ PageKeyedDataSource Ϊ��

����һ������Դ�� ���� Language Ϊ demo �е�ʵ�����

class LanguageDataSource: PageKeyedDataSource<Int, Language>()
ʵ������ override ����

override fun loadInitial(params: LoadInitialParams<Int>, callback: LoadInitialCallback<Int, Language>) {
}
override fun loadAfter(params: LoadParams<Int>, callback: LoadCallback<Int, Language>) {
}
override fun loadBefore(params: LoadParams<Int>, callback: LoadCallback<Int, Language>) {
}
�� 3 �����������ν���Ϊ

���μ���
����һҳ����
ǰһҳ����
���Ǹ���һҳ��������߼�

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
���� LanguageRepository ������ retrofit ������һ�� Language ������б� ���ǵ��� callback.onResult �ͻ�ˢ�� RecyclerView ����ͼ

loadAfter ��ʵ�ִ����� loadInitial һ�£����ﲻ��׸����

����������һ�� UI �㣬���Ƕ���һ�� PagedListAdapter

class LanguageAdapter(private val context: Context) : PagedListAdapter<Language, ViewHolder>(languageDiff)
����������Ҫ override 2������

 override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder
override fun onBindViewHolder(holder: ViewHolder, position: Int)
�� onBindViewHolder �У� ���ǿ���ͨ�� getItem(position) ��ȡ����ڵ�����ʵ��ȥ���� UI ��չʾ��

��������һ���ȽϹؼ��Ĳ��֣��Ǿ���������� DATA �� UI �������֡�

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
����� pageList �� NotifyExecutor �� FetchExecutor Ҳ�Ǳ������õġ��� Android arch componet �����ļܹ��У����Ƽ�ʹ�ù���һ�� PageList �� LiveData �ķ�ʽ�����ǲ�ʹ��Ҳû�й�ϵ��arch compoent ���������������ﲻ��������������������ϸʹ�ÿ��Բ鿴google��ʵ��Դ��

�ڴ����˽��� paging ����ɲ��ֺ����ǻῪʼ���棬�����ǵ���Ϊʲô��Ҫ paging �أ� ��������֮ǰ��ͨ��ʹ�÷�ʽ��ʲô�����أ����ǿ�����Դ����Ѱ�ҵ��𰸡�

���ǿ����� 2 �����ֵ������ԽӴ���Ϊ�������з������鿴 PagedList.Builder#build() ��Դ�룺

return PagedList.create(
    mDataSource,
    mNotifyExecutor,
    mFetchExecutor,
    mBoundaryCallback,
    mConfig,
    mInitialKey);
�����鿴

return new ContiguousPagedList<>(contigDataSource,
    notifyExecutor,
    fetchExecutor,
    boundaryCallback,
    config,
    key,
    lastLoad);
���������Ĺ��췽�������Կ��������߼�

mDataSource.dispatchLoadInitial(key,
    mConfig.initialLoadSizeHint,
    mConfig.pageSize,
    mConfig.enablePlaceholders,
    mMainThreadExecutor,
    mReceiver);
������ PageKeyedDataSource Ϊ���� ������ DataSource ����ͬ��

�鿴 dispatchLoadInital ����

LoadInitialCallbackImpl<Key, Value> callback =
                new LoadInitialCallbackImpl<>(this, enablePlaceholders, receiver);
loadInitial(new LoadInitialParams<Key>(initialLoadSize, enablePlaceholders), callback);

callback.mCallbackHelper.setPostExecutor(mainThreadExecutor);
�������ǿ��Կ����� loadInitial ����������Ҫ�� override �ķ���֮һ��������������� callback �� onResult �������׷�����ʲô�أ�

�鿴 LoadInitialCallbackImpl#onResult() ��Դ�룬�ؼ��߼�����

mDataSource.initKeys(previousPageKey, nextPageKey);
int trailingUnloadedCount = totalCount - position - data.size();
if (mCountingEnabled) {
    mCallbackHelper.dispatchResultToReceiver(new PageResult<>(
    data, position, trailingUnloadedCount, 0));
} else {
    mCallbackHelper.dispatchResultToReceiver(new PageResult<>(data, position));
}
�鿴 dispatchResultToReceiver

image

�����鿴 onPageResult ����

image

���ǹ�עһ�� init ʱ����߼�

mStorage.init(pageResult.leadingNulls, page, pageResult.trailingNulls,
                        pageResult.positionOffset, ContiguousPagedList.this);
init ���߼��ܼ򵥣�ֻ�� 2 ��

init(leadingNulls, page, trailingNulls, positionOffset);
callback.onInitialized(size());
image

����� ���ǿ��Կ����ؼ����߼�

mPages.clear();
mPages.add(page);
����� PageList �󶨵����ݾͷ����˱仯��֮�����ǰ� PageList submit ���� adapter ��ô�����ݾͷ����˸��¡�

��ʼ�������ǿ����ˣ���ô��ʣ�µ���������μ��ص���

���Ƿ������� RecyclerView�� ������ǻ����б��������������ʱ�򣬺���Ȼ����� adapter �� bind ��������ô������ȥ�鿴 PagedListAdapter#getItem ��Դ�롣

return mDiffer.getItem(position);
image

�鿴 PageList �� loadAround

loadAroundInternal(index);
������

image

if (mAppendItemsRequested > 0) {
    scheduleAppend();
}
�鿴 scheduleAppend ��ʵ��

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
������ǿ����� dispatchLoadAfter �����ĵ��ã�֮����߼���֮ǰ�� dispathLoadInitial �ͷǳ��������ˡ�

���գ�����õ������߼� image

������� AsyncPagedListDiffer �� PagedList.Callback �Ļص�

image

���callback �Ǻ� adapter ���������ġ����Ի�������ˢ���б�

������ǿ�һ�� Adapter �� submit �����������Կ����������߼� image

���ǿ��Կ��� paging �������� DiffUtils �� RecyclerView ����ˢ�µġ���������Ҳ���赣�� paging ������������⡣

���
���̸һ�¶� paging ����⡣ һ������£�������ԭʼ�ķ�ʽ���б� UI ���ڵĲ��֣�����Ҫ֪�����ݵ���Դ���߼����֣������ڳ����� mvp ģʽ�У�������ݺ� UI ���зֲ㡣 �� paging ������һϵ�еķ�װ�� �ṩ�˸���ͨ�õ� API ����������Щ���顣��ͨ�׵�˵������ʵ���˷�ҳ���ؽṹ�е� Presenter �㼰 Presenter������δ����֡�

����ģʽ��ҵ��ı�д�ߣ����԰� UI ���ֵĴ���ģ�廯�� ֻ��Ҫ����ҵ���߼������Ұ�ҵ���߼��е����ݻ�ȡд�� DataSource �У�ʹ��ҳ���صĲ�������̶ȸ��ߡ�