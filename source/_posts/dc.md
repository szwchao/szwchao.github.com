title: dc
date: 2015-12-10 15:49:48
tags: [work, dc]
categories: work
toc: true
---

## DC类图

{% plantuml %}
    ' 放大1.5倍
    ' scale 1.5
    ' 定义类图的背景，边框已经箭头的颜色
    skinparam class {
        ' BackgroundColor #E6FFCC
        ArrowColor black
        BorderColor black
    }
    title : DC classes
    class TaskCtrl {
        std::list<SubTask*> mSubTaskList;
        void InitSubTasks(void);
        virtual void RunSubTasks(void) = 0;
        void AddSubTask(SubTask*);
        virtual void ReqRun(SubTask*){};
    }
    class TaskCtrlEvent extends TaskCtrl {
       -mSubTaskQueue
       -mSemaSubTaskQueue
       +TaskCtrlEvent(void)
       +~TaskCtrlEvent(void)
       +RunSubTasks(void)
       +ReqRun(SubTask*)
    }
    class TaskCtrlPeriodic extends TaskCtrl {
       +RunSubTasks(void)
    }
    
    class SubTask {
       #mpTaskCtrl
       +InitSubTask(void)
       +RunSubTask(void)
       +SetTaskCtrlPointer(TaskCtrl*)
       #ReqTaskTime(void)
    }
    
    class Observer {
       +SubscribtionCancelled(Subject* pSubject)
       +Update(Subject* pSubject)
       +SetSubjectPointer(int id, Subject* pSubject)
       +ConnectToSubjects(void)
       +SetObserverId(int id)
       +GetObserverId()
       -mId
    }
    
    class SwTimerBaseClass extends Observer {
       +SubscribtionCancelled(Subject* pSubject)
       +Update(Subject* pSubject)
       +SetSubjectPointer(int Id,Subject* pSubject)
       +ConnectToSubjects()
       +mpTimerObjList
       +mNumbersOfElements
    }
    
    class ISubject {
       +GetSubjectId(void)
       +GetSubjectType(void)
       +GetRefCount(void)
       +Subscribe(Observer* pObserver)
       +SubscribeE(Observer* pObserver)
       +Unsubscribe(Observer* pObserver)
    }
    
    class Subject extends ISubject {
       -mObserverList
       -mObserverListE
       #mId
       #mType
       #mRefCount
       #mDestroyed
       +SetSubjectId(SUBJECT_ID_TYPE id)
       +GetSubjectId(void)
       +GetSubjectType(void)
       +SetSubjectType(SUBJECT_TYPE type)
       +IncRefCount()
       +GetRefCount()
       +Subscribe(Observer* pObserver)
       +SubscribeE(Observer* pObserver)
       +Unsubscribe(Observer* pObserver)
       +GetFlashId(void)
       +SaveToFlash(IFlashWriter* pWriter, FLASH_SAVE_TYPE save)
       +LoadFromFlash(IFlashReader* pReader, FLASH_SAVE_TYPE savedAs)
       #NotifyObservers(void)
       #NotifyObserversE(void)
    }
    
    class SubjectPtr {
       +IsValid()
       +Attach(Subject* pSubject)
       +Detach(void)
       +Detach(ISubject* pSubject)
       +Equals(ISubject* pSubject)
       +Subscribe(Observer* pObserver)
       +Unsubscribe(Observer* pObserver)
       +UnsubscribeAndDetach(Observer* pObserver)
       +GetSubject(void)
       +operator ->()
       +Update(ISubject* pSubject)
       +IsUpdated(bool reset = true)
       +ResetUpdated(void)
       +SetUpdated(void)
       -m_pSubject
       -m_updated
    }
    
    class app extends SwTimerBaseClass, SubTask {
    }
{% endplantuml %}

<!-- more -->

## Main

DC的main里建了5个Task：

* ControllerPeriodicTask
* ControllerEventsTask
* GeniAppTask
* LowPrioPeriodicTask
* DisplayEventTask

之后会调用RunFactory().

{% plantuml %}
scale 1.5
:WATCH_DOG_INIT();
:InitUsbDriverStack();
:OS_InitKern();
:IobComDrv::GetInstance()->InitInterComTask();
note right : OS_CREATETASK(&mTCBInterComTask, "InterCom Task", InterComTask, 200, mStackInterComTask);
:SwTimerTask::GetInstance();
:MPCIPConfig::createObjects();
note right : 实现一些IP协议，webserver
:OS_InitHW();
:EnablePowerDownIrq();
:DisplayInit();
:ConfigControl::GetInstance()->Init();
note right : 操作flash，一些配置以及log
partition 开始创建Task {
    :OS_CREATETASK(ControllerPeriodicTask);
    :OS_CREATETASK(ControllerEventsTask);
    :OS_CREATETASK(GeniAppTaskStack);
    :OS_CREATETASK(LowPrioPeriodicTask);
    :OS_CREATETASK(DisplayEventTask);
}
:RunFactory();
:GeniSysInit();
partition 各任务初始化子任务 {
    :GetGeniAppTask()->InitSubTasks();
    :GetControllerEventsTask()->InitSubTasks();
    :GetControllerPeriodicTask()->InitSubTasks();
    :GetLowPrioPeriodicTask()->InitSubTasks();
    :GetDisplayEventsTask()->InitSubTasks();
}
:OS_Start();
{% endplantuml %}

## Subject

属性：

```cpp
std::vector<Observer*> mObserverList;
std::vector<Observer*> mObserverListE;
SUBJECT_ID_TYPE mId;
SUBJECT_TYPE mType;
int mRefCount;
bool mDestroyed;
```

> mId在数据库中表Subject里定义了id，name，type，是否是GeniAppIf，save（？），FlashBlock（？），Verified

> mType在数据库中表SubjectTypes里定义了Type


方法：

```cpp
virtual void SetSubjectId(SUBJECT_ID_TYPE id);
virtual SUBJECT_ID_TYPE GetSubjectId(void);

virtual SUBJECT_TYPE GetSubjectType(void);	
virtual void SetSubjectType(SUBJECT_TYPE type);	

void IncRefCount();	// may ONLY be called by the SubjectPtr::Attach function
int GetRefCount();
  
virtual void Subscribe(Observer* pObserver);
virtual void SubscribeE(Observer* pObserver);
virtual void Unsubscribe(Observer* pObserver);

virtual void NotifyObservers(void);
virtual void NotifyObserversE(void);
```

Factory.cpp里有`SUBJECTS[]`，由数据库Factory.mdb表Subject生成，每个item包含id，type。

```cpp
const DbSubject SUBJECTS[] = {
  {1, SUBJECT_TYPE_INTDATAPOINT}, // display_contrast
  {2, SUBJECT_TYPE_INTDATAPOINT}, // display_backlight
   ......
}
```

再根据不同的type，在数据库Factory.mdb的相应子表中定义，如FloatDataPoint类型的，则有表FloatDataPoint，如果这张表里有相应的datapoint定义，则在代码里将这个datapoint初始化成相应的最小最大值和初始值，没有就按default来初始化（？）。

RunFactory()遍历`SUBJECTS[]`，初始化，并调用`StoreSubject()`，用**Subject**的类函数`SetSubjectId`和`SetSubjectType`，存入`subject_map`这样一个`std::map`里。
## Observer

### 建立Observer

```cpp
struct DbObserver
{
  U16 Id;
  OBSERVER_TYPE TypeId;
  int SubjectId;
};
const DbObserver OBSERVERS[] = {
  {1, ObTypeDisplayController, -1}, // display_controller
  ......
}
```
RunFactory()遍历`OBSERVERS[]`，由其TypeId调相应的`ConstructObserver(observerId)`

Observer表里:

| Id | Name            | TypeId        | ConstructorArgs      | TaskId | SubjectId | TaskOrder | Comment |
|----|-----------------|---------------|----------------------|--------|-----------|-----------|---------|
| 5  | status_led_ctrl | StatusLedCtrl | ControllerEventsTask |        |           |           |         |
							
							
ConstructObserver()里：

sql语句为：
```sql
SELECT Observer.Id, Observer.Name, ObserverType.NameSpace, ObserverType.Name, Observer.ConstructorArgs FROM Observer, ObserverType WHERE (Observer.TypeId=ObserverType.Id) AND (ObserverType.IsSingleton=0) ORDER BY Observer.Id
```

```cpp
static Observer* ConstructObserver(const U16 observerId)
{
  switch (observerId)
    case 5:  //status_led_ctrl
        return new StatusLedCtrl();
}
```

如果要加一个模块，需先在`ObserverType`表里新加一行，name为模块的类名（构造函数）。

然后在`Observer`表里新加一行，`TypeId`选`ObserverType`里新加行的**name**，设定**TaskId**

RunFactory()里根据不同的TypeId构建不同的Observer

```cpp
Observer* p_observer;
for (j = 0; j < OBSERVERS_CNT; j++)
{
  switch(OBSERVERS[j].TypeId)
    case ObTypeStatusLedCtrl:
      p_observer = ConstructObserver(OBSERVERS[j].Id);
    break;
    ......
  if (p_observer) {
    StoreObserver(OBSERVERS[j], p_observer);   //存到observer_map里
  } else {
    FatalErrorOccured("Factory generator: Unknown observer!");
    while(1); // Unknown observer!
  }
}
```

### 建立Observer与Task的关联

OBSERVER_TASK的结构：

```cpp
struct DbObserverTask
{
  U16 ObserverId;
  U16 TaskId;
};
```

由该条SQL语句生成：

```sql
SELECT Observer.Id, Task.Id FROM Observer, Task WHERE (((Observer.TaskId)=[Task].[Id])) ORDER BY Task.Id, Observer.TaskOrder, Observer.Id
```

建立联系：

```cpp
for (j = 0; j < OBSERVER_TASK_CNT; j++)
{
  p_sub_task = dynamic_cast<SubTask*>(GetObserver(OBSERVER_TASK[j].ObserverId)); //相应的Observer强制转为SubTask
  p_task = GetTask(OBSERVER_TASK[j].TaskId);  //返回5个主Task之一
  if (p_sub_task && p_task) {
    p_task->AddSubTask(p_sub_task);           //主Task添加其为子Task
    p_sub_task->SetTaskCtrlPointer(p_task);   //子Task也添加父Task
  } else {
    FatalErrorOccured("observer task");
    while (1);
  }
}
```

### 建立Observer与Subject的联系（Observer挂接Subject）

首先要在数据库中建立联系，表ObserverSubjects里，形如：

| SubjectId                    | ObserverId         | SubjectRelationId     | id   | Access | Comment |
|------------------------------|--------------------|-----------------------|------|--------|---------|
| operation_mode_actual_pump_6 | advanced_flow_ctrl | OPERATION_MODE_PUMP_6 | 6923 | Read   |         |
| operation_mode_actual_pump_5 | advanced_flow_ctrl | OPERATION_MODE_PUMP_5 | 6922 | Read   |         |

OBSERVER_SUBJECTS的结构：

```cpp
struct DbObserverSubject
{
  U16 SubjectId;
  U16 ObserverId;
  int SubjectRelation;
};
```

由该条SQL语句生成：

```sql
SELECT ObserverSubjects.SubjectId AS SubjectId, Observer.Id AS ObserverId, 'SP_'+ObserverType.ShortName+'_'+SubjectRelation.Name AS SubjectPointerName FROM ObserverType INNER JOIN ((Observer INNER JOIN SubjectRelation ON Observer.TypeId = SubjectRelation.ObserverTypeId) INNER JOIN ObserverSubjects ON (SubjectRelation.Id = ObserverSubjects.SubjectRelationId) AND (Observer.Id = ObserverSubjects.ObserverId)) ON ObserverType.Id = Observer.TypeId ORDER BY ObserverId, ObserverSubjects.SubjectId
```

建立联系:

```cpp
Subject* p_subject;
int refCount;
for (j = 0; j < OBSERVER_SUBJECTS_CNT; j++)
{
  p_observer = GetObserver(OBSERVER_SUBJECTS[j].ObserverId);
  p_subject = GetSubject(OBSERVER_SUBJECTS[j].SubjectId);
  if (p_observer && p_subject)
  {
    refCount = p_subject->GetRefCount();
    //这里是各个Observer的SubjectPtr成员变量与相应的Subject挂接
    p_observer->SetSubjectPointer(OBSERVER_SUBJECTS[j].SubjectRelation,p_subject);
    //这里refCount应当自加过了，否则就是出错了
    if (refCount == p_subject->GetRefCount())
    {
      // observer did not set the subject pointer
      if (p_observer) p_observer = p_observer; // ease debug
      if (p_subject) p_subject = p_subject;  // ease debug
     FatalErrorOccured("SetSubjectPointer");
     while (1);
    }
  } else {
   FatalErrorOccured("Subject Observer");
    while (1);
  }
}
```

`SetSubjectPointer`是Observer的纯虚函数，在各个app里实现。实现形式为：

```cpp
void DDACtrl::SetSubjectPointer(int id, Subject* pSubject)
{
  switch (id)
  {	
	case SP_DDAC_FIXED_DOSING_ENABLED:
	  mpDDAed.Attach(pSubject);
	  break;
	case SP_DDAC_DOSING_QUANTITY_ACT:
	  mpDosingQuantity.Attach(pSubject);
	  break;
	default:
      break; 
  }
}
```
mpDDAed为SubjectPtr类，SubjectPtr相当于Subject的指针类，Attach的作用就是把pSubject强制转换为该类的成员变量m_pSubject（模板类），同时自加pSubject的refCount;

### 建立Observer与Subject的联系（Subject挂接Observer）

对每一个Observer，调用`ConnectToSubjects()`

```cpp
for (OBSERVER_MAP_ITR itr = observer_map.begin(); itr != observer_map.end(); itr++)
{
  itr->second->ConnectToSubjects();
}
```

ConnectToSubjects()由各个app实现:

```cpp
void DDACtrl::ConnectToSubjects()
{
  mpDDAed->Subscribe(this);
  mpDosingQuantity->Subscribe(this);
}
```

mpDDAed为SubjectPtr类，其`Subscribe`函数其实调用的Subject类的Subscribe函数:

```cpp
void Subject::Subscribe(Observer* pObserver)
{
  OS_EnterRegion();
  mObserverList.push_back(pObserver);
  OS_LeaveRegion();
}
```

对每个Subject都有成员mObserverList，这个函数将对应的Observer放入这个列表里。

至此就建立了每个Observer与Subject的双向联系。
