# Struct std::sync::Mutex 
A mutual exclusion primitive useful for protecting shared data

This mutex will block threads waiting for the lock to become available.   
## 创建 
The mutex can also be statically initialized or created via a new constructor.  

## 参数
Each mutex has a type parameter范围 which represents the data that it is protecting.  

## 使用
The data can only be accessed through the RAII guards returned from lock and try_lock, which guarantees that the data is only ever accessed when the mutex is locked.   

## 返回  Result


## 原理 Poisoning
The mutexes in this module implement a strategy called “poisoning” where a mutex is considered poisoned whenever a thread panics while holding the mutex.    
Once a mutex is poisoned, all other threads are unable to access the data by default as it is likely tainted (some invariant is not being upheld). 

For a mutex, this means that the lock and try_lock methods return a Result which indicates whether a mutex has been poisoned or not.    
Most usage of a mutex will simply unwrap() these results, propagating panics among threads to ensure that a possibly invalid invariant is not witnessed.

### 例外  
A poisoned mutex, however, does not prevent all access to the underlying data.     
The PoisonError type has an into_inner method which will return the guard that would have otherwise been returned on a successful lock.    
This allows access to the data, despite the lock being poisoned.