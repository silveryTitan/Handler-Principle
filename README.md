# <p align="center">Handler原理及相关知识点</p>
#### 它的使用场景是把任务切换到需要的线程去执行，一般常用场景是更新UI，在ViewRootImpl类中做了当前线程是否是主线程的处理，所以无法在子线程中更新UI。因为UI控件不是线程安全的，多线程操作会使UI控件处于不可预期的情况，线程不安全的处理方式是对UI控件进行加锁操作，但加锁的缺点是会让UI的访问逻辑变得复杂并降低了访问效率并会阻塞某些线程的执行。
#### ViewRootImpl是一个视图层次结构的顶部，实现了View与WindowManager之间的协议，它的创建是在WindowManagerLobal.addview时创建的。

#### Handler的运行需要底层的MessageQueue和Looper的支撑。
#### MessageQueue的存储结构是单链表结构，每个节点都包含了值域和指针域，指针域指向下一个节点，最后一个节点指向空值。
#### Looper会无限循环的查看队列中有没有新消息，有就处理没有就等待。线程默认是没有Looper的而UI线程在被创建的时候就已经初始化了，所以可以直接在UI线程直接使用。
#### Looper的初始化是在应用启动时由ActivityManagerService中的Process.start方法找到ActivityThread的main函数并执行创建。
#### 每一个Looper分别对应了一个Thread和MessageQueue，因为Looper和Thread是一对一的对应关系，所以Handler和Thread也是一对一的，但是两者并没有实际的关系。
#### Looper使用了ThreadLocal进行保存，确保了每个线程只有一个Looper。
#### ThreadLocal是一个线程内部的存储类，它的侧重点是数据隔离，原理是以空间换空间的方式为每个线程都提供一个变量的副本，从而实现了数据的互不干扰，它的静态方法内部类ThreadLocalMap为每个Thread都维护了一个数组table，Thread确定了一个数组下标，这个下标就是value存储对应的位置。

#### Handler的工作原理是发消息向MessageQueue插入一个消息，messageQueue的next方法会将消息给looper，looper在交给handler处理，handler的分发消息被调用判断message的callback是否为空不为空就交给handler处理，为空就交给handler的handlerMessage方法处理消息。
#### Messagequeue主要包含两个操作，插入enqueueMessage和删除next，插入方法是将所有消息按时间进行排序，删除方法是当前队列存在待处理消息就将消息出队并赋值下一条消息，否则就阻塞。
 
#### 如果想在子线程使用Handler可以使用HandlerThread,因为它继承了Thread，所以本质是线程，在run方法中创建了Looper并加锁保证了线程的安全。
#### IntentService就是在子线程中执行逻辑的封装抽象类，在onCreate创建HandlerThread，当Service生命周期执行完系统就会销毁service，需要在onDestory中退出looper
