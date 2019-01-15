### add,remove����������
ʹ��FragmentTransaction��add��remove����������
```
getSupportFragmentManager().beginTransaction()
                    .add(R.id.fl_container,emptyFragment)
                    .commit();

getSupportFragmentManager().beginTransaction()
                            .remove(emptyFragment)
                            .commit();
```
�о���activity������
```
01-05 12:50:13.694 25984-25984  D/EmptyFragment: onCreate
01-05 12:50:13.707 25984-25984  D/EmptyFragment: onCreateView
01-05 12:50:13.708 25984-25984  D/EmptyFragment: onStart
01-05 12:50:13.708 25984-25984  D/EmptyFragment: onResume
01-05 12:50:18.896 25984-25984  D/EmptyFragment: onPause
01-05 12:50:18.896 25984-25984  D/EmptyFragment: onStop
01-05 12:50:18.896 25984-25984  D/EmptyFragment: onDestroyView
01-05 12:50:18.899 25984-25984  D/EmptyFragment: onDestroy
```

### onHiddenChanged��ʱ����
ʹ��FragmentTransaction��show��hide���ǻᴥ�� onHiddenChanged
FragmentTransaction.add
```
01-05 12:50:13.694 25984-25984  D/EmptyFragment: onCreate
01-05 12:50:13.707 25984-25984  D/EmptyFragment: onCreateView
01-05 12:50:13.708 25984-25984  D/EmptyFragment: onStart
01-05 12:50:13.708 25984-25984  D/EmptyFragment: onResume
```
��ʱ��Fragment�Ѿ���ʾ������
FragmentTransaction.hideֻ�����onHiddenChanged
```
01-05 13:02:46.384 26851-26851  D/EmptyFragment: onHiddenChanged  true
```
�ٵ���FragmentTransaction.show
```
01-05 13:02:46.981 26851-26851  D/EmptyFragment: onHiddenChanged 0 false
```

### setUserVisibleHint��ʱ����

> An app may set this to false to indicate that the fragment's UI is scrolled out of visibility or is otherwise not directly visible to the user

֮ǰ��add,remove,show,hide,�����ᴥ�����.������Ǻܳ��õ�,�ٶ����¾���ʹ��FragmentPagerAdapterʱ�����
�����������
```
vp.setAdapter(new FragmentPagerAdapter(getSupportFragmentManager()) {
            @Override
            public Fragment getItem(int i) {
                if (i==  1){
                    return EmptyFragment2.getInstance(i+"");
                }
                if (i == 2){
                    return EmptyFragment3.getInstance(i+"");
                }
                return EmptyFragment.getInstance(i+"");
            }

            @Override
            public int getCount() {
                return 3;
            }
        });
```
�����EmptyFragment���ǿյ�,ֻ�Ǵ�ӡ���¸��Ե��������ں�������.
```
01-05 13:22:35.488 28123-28123  D/EmptyFragment: setUserVisibleHint null false
01-05 13:22:35.488 28123-28123  D/EmptyFragment1: setUserVisibleHint null false
01-05 13:22:35.488 28123-28123  D/EmptyFragment: setUserVisibleHint null true
01-05 13:22:35.488 28123-28123  D/EmptyFragment: onCreate 0
01-05 13:22:35.488 28123-28123  D/EmptyFragment1: onCreate 1
01-05 13:22:35.492 28123-28123  D/EmptyFragment: onCreateView 0
01-05 13:22:35.493 28123-28123  D/EmptyFragment: onStart 0
01-05 13:22:35.493 28123-28123  D/EmptyFragment: onResume 0
01-05 13:22:35.495 28123-28123  D/EmptyFragment1: onCreateView 1
01-05 13:22:35.496 28123-28123  D/EmptyFragment1: onStart 1
01-05 13:22:35.496 28123-28123  D/EmptyFragment1: onResume 1
```
�����и���,setUserVisibleHint����onCreate ����,���Ǵ�����һ����onCreate�����,����setUserVisibleHint �л��и�null
���о���setUserVisibleHint ���������,��һ����Ϊfalse,�ڶ�����Ϊtrue
```
  public static EmptyFragment getInstance(String position){
        EmptyFragment emptyFragment = new EmptyFragment();
        Bundle bundle = new Bundle();
        bundle.putString("position",position);
        emptyFragment.setArguments(bundle);
        return emptyFragment;
    }


    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        position = getArguments().getString("position");
        Log.d(TAG,"onCreate " + position);
    }

```
ViewPager��Ĭ�ϳ�ʼ������Fragment��,�����Ҹ�������Fragment,�ٻ���һ�¿���,��ǰ�������ڶ�ҳ
```
01-05 13:27:38.474 28123-28123  D/EmptyFragment2: setUserVisibleHint null false
01-05 13:27:38.474 28123-28123  D/EmptyFragment: setUserVisibleHint 0 false
01-05 13:27:38.474 28123-28123  D/EmptyFragment1: setUserVisibleHint 1 true
01-05 13:27:38.474 28123-28123  D/EmptyFragment2: onCreate 2
01-05 13:27:38.479 28123-28123  D/EmptyFragment2: onCreateView 2
01-05 13:27:38.480 28123-28123  D/EmptyFragment2: onStart 2
01-05 13:27:38.480 28123-28123  D/EmptyFragment2: onResume 2
```
EmptyFragment�� setUserVisibleHint��Ϊfalse��,�Ҳ���Ҳ�õ���, EmptyFragment1��setUserVisibleHint ��Ϊtrue��,EmptyFragment2��ʼ��ʼ����
����:
setUserVisibleHint ��ʹ����Viewpager�е�
1. setUserVisibleHint������onCreate���õ�
1. Fragment��ʼ����ʱ����Ĭ����Ϊfalse,Ȼ���������Ҫ��ʾ����Ϊtrue
1. �ۺ�1��2,setUserVisibleHint�в�Ҫ����������صĲ���,ֻ��������ʾ��־λ�Ϳ�����

### ���
Fragment���ӳټ���ò��ֻ������ViewPager��
1. ͨ��setUserVisibleHint ���жϵ�ǰ�Ƿ�����ʾFragment,һ����־λ
1. ͨ��onResume���ж�Fragment�Ƿ��Ѿ���ʼ�����,�ڶ�����־λ
1. ��������־λ��Ϊtrue��ʱ��,��������,������һ����־λ,���ݼ�����ɵı�־λ.