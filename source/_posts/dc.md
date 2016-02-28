title: dc
date: 2015-12-10 15:49:48
tags: [work, dc]
categories: work
toc: true
---

# Head 1

> 摘要文字

[Baidu](http://www.baidu.com)

![](/images/autosar.png)

## Head 2

list 1:

* item 1
* item 2
* item 3
* item 4

## Head 2

```python
@requires_authorization
class SomeClass:
    pass

if __name__ == '__main__':
    pass
```

### Head 3
#### Head 4
##### Head 5
###### Head 6

| table | l  | l  |
|-------|----|----|
| c     | rr | er |
| w     | er | qq |

### Test plantuml
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

```
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes 
or No?|approved:>http://www.baidu.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```
