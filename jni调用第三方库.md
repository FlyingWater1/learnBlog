## ��̬��͹����

��ʹ�÷����Ϸֿ�����Ͽ��Է�Ϊ���ࣺ��̬��͹���⡣��windows�о�̬������ .lib Ϊ��׺���ļ������������.dll Ϊ��׺���ļ�����linux�о�̬������ .a Ϊ��׺���ļ������������ .soΪ��׺���ļ���

�������뾲̬������ʱ������Ŀ���ļ����������н�������ʹ�õĺ����Ļ����뱻copy�����յĿ�ִ���ļ��С���ͻᵼ���������ɵĿ�ִ�д�������Ա�࣬�൱�ڱ����������벹�������ˣ���������������ԾͿ�Щ���������и�ȱ��: ռ�ô��̺��ڴ�ռ�. ��̬��ᱻ��ӵ��������ӵ�ÿ��������,������Щ��������ʱ, ���ᱻ���ص��ڴ���. �������ֶ������˸�����ڴ�ռ�.

�빲������ӵĿ�ִ���ļ�ֻ��������Ҫ�ĺ��������ñ����������еĺ������룬ֻ���ڳ���ִ��ʱ,��Щ��Ҫ�ĺ�������ű��������ڴ��С�������ʹ��ִ���� ���Ƚ�С,��ʡ���̿ռ䣬����һ��������ϵͳʹ�������ڴ棬ʹ��һ�ݹ����פ�����ڴ��б��������ʹ�ã�Ҳͬʱ��Լ���ڴ档������������ʱҪȥ���ӿ�Ứ ��һ����ʱ�䣬ִ���ٶ���Ի���һЩ���ܵ���˵��̬���������˿ռ�Ч�ʣ���ȡ��ʱ��Ч�ʣ��������������ʱ��Ч�ʻ�ȡ�˿ռ�Ч�ʣ�û�к��뻵������ֻ�� ������Ҫ�ˡ�'


CMake���ĵ����Կ� https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories

��googleDemo��
```
 sourceSets {
        main {
            // let gradle pack the shared library into apk
            jniLibs.srcDirs = ['../distribution/gperf/lib']
        }
    }
```
�뵽�˶�̬������Ҫ�������apk�е�,demo��ʹ���˾�̬��͹����,��ٶȵ�ͼ�ȵ��������Ǹ�.so�ļ�,����һ��һ����

### ���ù����
1 ��hello-libs�Ŀ⿽����������Ŀ��appĿ¼��
2 ��app��build.gradle�м���,������ٶȵ�ͼ��soһ����
```
sourceSets {
        main {
            // let gradle pack the shared library into apk
            jniLibs.srcDirs = ['../distribution/gperf/lib']
        }
    }
```
3 �޸�CMakelist.txt,����������ݼӽ�ȥ
3.1 ����lib_gperf����
3.2 ����lib_gperf���λ��
3.3 ����lib_gperfͷ�ļ�
3.4 ��lib_gperf���ӵ�native-lib��
```

# configure import libs
# ����һ��������distribution_DIR ,ֵ�Ǻ����
set(distribution_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../distribution)

# shared lib will also be tucked into APK and sent to target
# refer to app/build.gradle, jniLibs section for that purpose.
# ${ANDROID_ABI} is handy for our purpose here. Probably this ${ANDROID_ABI} is
# the most valuable thing of this sample, the rest are pretty much normal cmake
# ���һ����lib_gperf�Ĺ����SHARED,  ��Դ�ǵ���ӿڿ� interface library
add_library(lib_gperf SHARED IMPORTED)

# ��lib_gperf ��������IMPORTED_LOCATION,������λ��
set_target_properties(lib_gperf PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/gperf/lib/${ANDROID_ABI}/libgperf.so)
		
.....
ԭ��������
.....

# ����include,ò�ƾ���ͷ�ļ�
target_include_directories(
        native-lib
        PRIVATE
        ${distribution_DIR}/gperf/include)


target_link_libraries( # Specifies the target library.
                       native-lib
                       lib_gperf
                       # Links the target library to the log library
                       # included in the NDK.
                        log )		


```
4 �����͹���������,ʣ�µľ�������ʹ����,дһ��C++����������
```
#include <gperf.h>

extern "C" JNIEXPORT jlong JNICALL
Java_www_lixiangfei_top_hello_1jni_MainActivity_testSo(
        JNIEnv *env,
        jobject instance/* this */) {
    uint64_t  time = GetTicks();
    return time;
}

```

### ���þ�̬��
1. ��̬�����.a֮ǰһ����ļ�����������
2. �޸�CMakeLists.txt ,���������������һ��
```
# ���һ����lib_gmath�ľ�̬��STATIC,  ��Դ�ǵ���ӿڿ� interface library
add_library(lib_gmath STATIC IMPORTED)
# ��lib_gmath ��������IMPORTED_LOCATION,������λ��
set_target_properties(lib_gmath PROPERTIES IMPORTED_LOCATION
        ${distribution_DIR}/gmath/lib/${ANDROID_ABI}/libgmath.a)
		# ����include,ò�ƾ���ͷ�ļ�
target_include_directories(
        native-lib
        PRIVATE
        ${distribution_DIR}/gperf/include
        ${distribution_DIR}/gmath/include)
		
target_link_libraries( # Specifies the target library.
                       native-lib
                       lib_gperf
                       lib_gmath
                       # Links the target library to the log library
                       # included in the NDK.
                        log )
```
4. ���Բ�����
```
#include <gmath.h>
extern "C" JNIEXPORT int JNICALL
Java_www_lixiangfei_top_hello_1jni_MainActivity_testStatic(
        JNIEnv *env,
        jobject instance/* this */) {
    int   result = gpower(2);
    return result;
}
```
����ͨ��~~~


NDK���ĵ�:https://developer.android.google.cn/ndk/guides/audio/
