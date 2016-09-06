## 2016/8/22 9:16:52 IPC
#### IPC��Inter-Process Communication�����̼�ͨ��
* Bundle
* �ļ�����
* AIDL
* Messenger
* ContentProvider
* Socket

##### Android�еĶ����ģʽ

��������£���Android�еĶ����ָ����һ��Ӧ���д��ڶ�����̵��������Android��ֻ��һ�ַ��������Ǹ��Ĵ������Activity��Service��Receiver��ContentProvider����AndroidMenifest��ָ��android:process���ԡ�
```xml
android:process=":remote"
```
�鿴������Ϣ��
```cmd
adb shell ps ������adb shell ps | grep packagename
```
 
**ʹ�ö��̻߳�������¼���������⣺**
* ��̬��Ա�͵���ģʽ��ȫʧЧ��
* �߳�ͬ��������ȫʧЧ��
* SharePreferences�Ŀɿ����½���
* Application���δ�����

##### IPC����֪ʶ

1. Serializable�ӿڣ�implements Serializable serialVersionUID���Զ�����hashֵ����ָ��1L���ܴ�̶��ϱ��ⷴ���л�ʧ�ܣ�  
2. Parcelable�ӿڣ�
3. **Binder**��ʵ����IBinder�ӿ�

##### Android�е�IPC��ʽ
1. Bundle����������ݱ����ܹ����л�������������͡�ʵ����Serializable�ӿڡ�Parcellable�ӿڵĶ����Լ�һЩAndroid֧�ֵ��������
2. ʹ���ļ���������ʹ���ı��ļ���XML�ȶ�/д˫��Լ�������ݸ�ʽ�����ڲ�����/д�����⡣ϵͳ��sharepreferences�ļ��Ķ�/д��һ���Ļ�����ԣ������ڴ��л���һ��sp�ļ��Ļ��棬����ڶ���̵�ģʽ�£�ϵͳ�����Ķ�/д�ͱ�ò��ɿ�������Ը߲����Ķ�/д���ʣ�sp�кܴ��ʻᶪʧ���ݣ���ˣ�**�������ڽ��̼�ͨ��ʹ��SharePreferences**��
3. ʹ��Messenger����������IPC�������ײ�ʵ����AIDL��
4. ��ǩ��Android��������̽����P80