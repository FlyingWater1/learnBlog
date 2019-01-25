AsyncTask��ϵͳ�ṩ���Է����л��̵߳���,ѧϰ������Ĵ�����Ϥ���߳��л�

�ٷ��ĵ�����ʾ��
```
 private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
     protected Long doInBackground(URL... urls) {
         int count = urls.length;
         long totalSize = 0;
         for (int i = 0; i < count; i++) {
             totalSize += Downloader.downloadFile(urls[i]);
             publishProgress((int) ((i / (float) count) * 100));
             // Escape early if cancel() is called
             if (isCancelled()) break;
         }
         return totalSize;
     }

     protected void onProgressUpdate(Integer... progress) {
         setProgressPercent(progress[0]);
     }

     protected void onPostExecute(Long result) {
         showDialog("Downloaded " + result + " bytes");
     }
 }
 
```
Ȼ���������
```
 new DownloadFilesTask().execute(url1, url2, url3);
```
����������������������,��������,�������
doInBackground�������߳�,onProgressUpdate��onPostExecute�����߳�,���Կ��Ժܷ����ʵ������ʱ�ڽ�������ʾ����

���캯��
```
    public AsyncTask() {
        this((Looper) null);
    }

	
    public AsyncTask(@Nullable Handler handler) {
        this(handler != null ? handler.getLooper() : null);
    }

    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     *
     * @hide
     */
    public AsyncTask(@Nullable Looper callbackLooper) {
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()
            : new Handler(callbackLooper);

        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }

```
Ĭ�Ϲ��캯��������mHandler����ͨ��mainLooper������
```
    private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler(Looper.getMainLooper());
            }
            return sHandler;
        }
    }
```
InternalHandler���Ǵ����߳��л������ߵĹؼ�,�����߳�Ҫ֪ͨ���̵߳�ʱ��,�ͷ�����ϢȻ��InternalHandler�͵��ö�Ӧ�ķ���
```
private static class InternalHandler extends Handler {
        public InternalHandler(Looper looper) {
            super(looper);
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
```
mWorker������������һ��,��������ɹ������̵�,�ȰѲ�����doInBackground,Ȼ��ȴ�ִ����ɺ�postResult,���̳���Callable�ӿ�,����ӿں�runnable�ӿں���,ֻ�����з���
```
private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
}

public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```

���и�mFuture,FutureTask�̳���Future��Runnable,��һ������ȡ�����첽������,����װһ��Callable��,��ʵ�ͺ�Thread,Runnable�Ĺ�ϵһ��
```
		mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
```
������д��done����,Ϊʲô?����������,�������ȡ����mWorker���ܲ�����postResut����,����AsyncTask��˵����û����,���Ҫ���ȷ�����Ƿ�������,��done�����и�get()
���get���ȡcallable�ķ���,postResultIfNotInvoked��������һ��,�׳����˾Ͳ�����,�������ȡ���˵Ļ�CancellationException��ץ��,Ȼ�����postResult(null),�������̾�������
���캯�������,������ĳ�ʼ����û���,��ĳ�ʼ��Ӧ�������static��Ա�����ȵ���,AsyncTask����static��Ա����
һ����һ�� ,�������,�����Ҫ��Ϊ�˲�Ҫ�ظ������߳�
```
	//�����̵߳Ĵ�С��С2,���4
    private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
    //����߳���,cpu������*2+1
	private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    //�߳̿��к�,�������
	private static final int KEEP_ALIVE_SECONDS = 30;

	//�̵߳Ĵ�������,���þ���Thread������
    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        public Thread newThread(Runnable r) {
            return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
        }
    };

	//����̵߳Ķ���,���128���ȴ�����,���һ�·�̫��,�ͱ�����
    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * An {@link Executor} that can be used to execute tasks in parallel.
     */
    public static final Executor THREAD_POOL_EXECUTOR;
	
	//�����̳߳�
    static {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
        threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
    }

	//����ÿ��ֻ��һ�������Executor,AsyncTask��exec���õľ�����
    /**
     * An {@link Executor} that executes tasks one at a time in serial
     * order.  This serialization is global to a particular process.
     */
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

    private static final int MESSAGE_POST_RESULT = 0x1;
    private static final int MESSAGE_POST_PROGRESS = 0x2;
	//��,Ĭ��Executor
    private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
    private static InternalHandler sHandler;
```
�ٿ���SerialExecutor,ÿ��execute��ʱ��,�ͷŵ�mTasks��,newһ��Runnable��װr,Ȼ��ŵ����е�β��,�����ǰ��mActivityΪ�յĻ��͵���scheduleNext,scheduleNext���ͷ��һ��runnable���������̳߳�����
sDefaultExecutor�������SerialExecutor��,���������Ǹ�static,�������jvm����һ��ִ����,���Ծ���ͨ�������ʵ����AsyncTask��˳��ִ��
Ϊ��Ҫ˳��ִ��,��������Ϊ��ֹ����ʹ��AsyncTaskִ�к�ʱ����,Ӱ�����ܰ�
```
    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
����ִ��,Ĭ�Ͼ����õ�SerialExecutor˳��ִ�е�,��������,������ֵ�����в���,Ȼ�󶪸�Exectorִ��
```
	public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
	
	public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }
```



