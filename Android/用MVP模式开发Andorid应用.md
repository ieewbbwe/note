# 用MVP模式开发Android应用

## 概述

MVP去年就很火了，但是担心掌握不好一直没有用。直到这一次面试面试官问了这个，当时对这个模式理解的不是很好，于是回来补习了一下。

学习之后我们要搞清楚几个问题：

1 什么是MVP模式？

2 他的出现解决了什么问题？

3 如何搭建一个MVP应用？

4 使用他会造成哪些新的问题？

### 1 什么是MVP模式

是一种软件架构模式，为了解耦，减少代码冗余。

Model 依然是业务逻辑和实体模型（M）

View 对应于Activity，负责View的绘制以及与用户交互(V)

Presenter 负责完成View于Model间的交互（P）

![](http://img.blog.csdn.net/20150309135723885?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdmVjdG9yX3lp/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

简单解释下这张图，View（也可以理解成Activity）中持有Presenter的实例，Presenter中持有View的接口和Model的实例。这就不难理解了，View和Model可以通过Presenter交互。并且，因为是接口，变换View也很容易，只要实现了IView接口，完全不会影响到Model的逻辑。这样就实现了解耦。

### 2 他的出现解决了什么问题

如果看明白了上图，就能明白它能做什么了。简而言之：解耦，减少冗余，灵活，结构清晰。

做过大型项目的哥们都知道，动辄都是几百上千的类，有的Activity中逻辑太多，后期看起来简直头疼。MVP的出现把Activity中的逻辑抽取出来，这样看起来就会简单很多。逻辑一目了然。

### 3 如何搭建一个MVP应用

我们这里以Google 的sample为例。

老司机自行fork：https://github.com/googlesamples/android-architecture 

我们以最简单的todo-mvp分支下的例子来讲怎么实现一个MVP应用。

首先来看一下他的项目和项目工程结构。

可以看出来是一个类似日历记事本的一个项目。功能比较简单。

添加任务；任务统计；按照状态切换任务；编辑任务。

我们就把首页挑出来分析一下。

**View(V)**

MVP中的View(V),负责绘制UI元素、与用户进行交互(在Android中体现为Activity),本例中指的就是首页这个TasksActivity;

他有显示所有任务；显示加载loading；显示筛选菜单；添加任务这些功能。google的工程师想的更加全面，还有根据任务ID跳转详情；snack在选择完成时的提示；在选择进行中的提示；在选择清空时的提示；首页TextView在没有任务时的文本；在切换不同Filter时的文本。还有很多不列举了，一会看一下接口里面定义的方法。将这些页面上有数据操作的功能都抽取出来放到IView接口中。

```
  interface View extends BaseView<Presenter> {

        void setLoadingIndicator(boolean active);

        void showTasks(List<Task> tasks);

        void showAddTask();

        void showTaskDetailsUi(String taskId);

        void showTaskMarkedComplete();

        void showTaskMarkedActive();

        void showCompletedTasksCleared();

        void showLoadingTasksError();

        void showNoTasks();

        void showActiveFilterLabel();

        void showCompletedFilterLabel();

        void showAllFilterLabel();

        void showNoActiveTasks();

        void showNoCompletedTasks();

        void showSuccessfullySavedMessage();

        boolean isActive();

        void showFilteringPopUpMenu();
    }
```

然后让TasksActivity中的TasksFragment实现这个接口，并实现上述定义的这些方法。V 就定义好了。

**Module(M)**

接下来看一下Module(M),负责存储、检索、操纵数据(有时也实现一个Model interface用来降低耦合);简单来说就是处理业务逻辑，本例中指得是TasksRepository，我们再来分析一下它会有哪些操作！

TasksDataSource接口中的定义，查找所有任务；根据ID查找任务；添加任务；编辑任务；删除任务；看下接口。

```
public interface TasksDataSource {

    interface LoadTasksCallback {

        void onTasksLoaded(List<Task> tasks);

        void onDataNotAvailable();
    }

    interface GetTaskCallback {

        void onTaskLoaded(Task task);

        void onDataNotAvailable();
    }

    void getTasks(@NonNull LoadTasksCallback callback);

    void getTask(@NonNull String taskId, @NonNull GetTaskCallback callback);

    void saveTask(@NonNull Task task);

    void completeTask(@NonNull Task task);

    void completeTask(@NonNull String taskId);

    void activateTask(@NonNull Task task);

    void activateTask(@NonNull String taskId);

    void clearCompletedTasks();

    void refreshTasks();

    void deleteAllTasks();

    void deleteTask(@NonNull String taskId);
}
```

通过上述对View和Module的接口定义不难看出来，View主要操作了一些UI上的显示，Module主要负责了数据上的逻辑处理。业务被很好的分层了。

**Presenter(P)**

最后我们来看一下Presenter(P),作为View与Model交互的中间纽带，处理与用户交互的负责逻辑。也可以理解为他促成了Module和View双方的通信。本例以TasksPresenter来解说。

在Presenter中要持有Module与View。看到这句话应该就能明白他是如何起到桥梁的作用了。

同样的，为了结构统一和以后的拓展性，Google工程师这里也采用了接口的定义方式。先贴出接口自己理解。

```
 interface Presenter extends BasePresenter {

        void result(int requestCode, int resultCode);

        void loadTasks(boolean forceUpdate);

        void addNewTask();

        void openTaskDetails(@NonNull Task requestedTask);

        void completeTask(@NonNull Task completedTask);

        void activateTask(@NonNull Task activeTask);

        void clearCompletedTasks();

        void setFiltering(TasksFilterType requestType);

        TasksFilterType getFiltering();
    }
```

我们以loadTasks为例来感受下他是如何发挥纽带作用的。

首先用户在Activity中下拉刷新会触发LoadTasks动作。

接下来会调用TasksPresenter的loadTasks方法（
这里的mPresenter就是TasksPresenter的实例）。我们这里只摘要核心代码，想看源码的去GIT里面看。

```

    /**
     * @param forceUpdate   Pass in true to refresh the data in the {@link TasksDataSource}
     * @param showLoadingUI Pass in true to display a loading icon in the UI
     */
    private void loadTasks(boolean forceUpdate, final boolean showLoadingUI) {
        if (showLoadingUI) {
            mTasksView.setLoadingIndicator(true);//显示加载的Loading
        }
        if (forceUpdate) {
            mTasksRepository.refreshTasks();//刷新任务
        }

        mTasksRepository.getTasks(new TasksDataSource.LoadTasksCallback() {//查找任务
            @Override
            public void onTasksLoaded(List<Task> tasks) {//请求成功的回调
                if (!mTasksView.isActive()) {
                    return;
                }
                if (showLoadingUI) {
                    mTasksView.setLoadingIndicator(false);
                }

                if (tasks.isEmpty()) {
                    // Show a message indicating there are no tasks for that filter type.
                    processEmptyTasks();
                } else {
                    // Show the list of tasks
                    mTasksView.showTasks(tasks);
                    // Set the filter label's text.
                    showFilterLabel();
                }
            }
        });
    }
```

大致逻辑就是通过Presenter中的Module实例调用查找任务，并在收到回调之后调用Presenter中的View 实例变换UI。

利用接口编程的好处就是灵活，比如说我这里想换一套UI但是业务逻辑不变，我只需要在写一个Activity并且实现IView接口即可，完全不用担心逻辑。修改逻辑也一样，只用增加接口方法，就可以达到逻辑的变换，不用担心UI受到影响。