title:  python mysql 分页程序
date: 2016-3-18 12:10:56
tags: python
category: dev
---

```python
import MySQLdb
import os
 
import hashlib
#DB parameter
strHost = 'localhost'
strDB = 'bigo_db'
strUser = 'root'
strPasswd = ''
def seEncode(ustr, encoding='utf-8'):
    '''负责把入数据库的字符串，转化成utf-8编码'''
    if ustr is None:
        return ''
    if isinstance(ustr, unicode):
        return ustr.encode(encoding, 'ignore')
    else:
        return str(ustr)
#connect to DB
def getConnect(db=strDB, host=strHost, user=strUser, passwd=strPasswd, charset="utf8"):
    return MySQLdb.connect(host=strHost, db=strDB, user=strUser, passwd=strPasswd, charset="utf8")
def initClientEncode(conn):
    '''mysql client encoding=utf8'''
    curs = conn.cursor()
    curs.execute("SET NAMES utf8")
    conn.commit()
    return curs
class MySQLQueryPagination(object):
    
    def __init__(self,conn,numPerPage = 20):
        self.conn = conn
        self.numPerPage = numPerPage
        
    def queryForList(self,sql,param = None):
        totalPageNum = self.__calTotalPages(sql,param)
        for pageIndex in range(totalPageNum):
            yield self.__queryEachPage(sql,pageIndex,param)
            
    def __createPaginaionQuerySql(self,sql,currentPageIndex):
        startIndex = self.__calStartIndex(currentPageIndex)
        qSql  = r'select * from (%s) total_table limit %s,%s' % (sql,startIndex,self.numPerPage)
        return qSql
    
    def __queryEachPage(self,sql,currentPageIndex,param = None):
        curs = initClientEncode(self.conn) 
        qSql = self.__createPaginaionQuerySql(sql, currentPageIndex)      
        if param is None:
            curs.execute(qSql)
        else:
            curs.execute(qSql,param)
            
        result = curs.fetchall()
        curs.close()
        return result
        
    def __calStartIndex(self,currentPageIndex):
        startIndex = currentPageIndex  * self.numPerPage;
        return startIndex;
    
    def __calTotalRowsNum(self,sql,param = None):
        ''' 计算总行数 '''
        tSql = r'select count(*) from (%s) total_table' % sql
        curs = initClientEncode(self.conn) 
        if param is None:
            curs.execute(tSql)
        else:
            curs.execute(tSql,param)
        result = curs.fetchone()
        curs.close()
        totalRowsNum = 0
        if result != None:
            totalRowsNum = int(result[0])
        return totalRowsNum
    
    def __calTotalPages(self,sql,param):
        ''' 计算总页数 '''
        totalRowsNum = self.__calTotalRowsNum(sql,param)
        totalPages = 0;
        if (totalRowsNum % self.numPerPage) == 0:
            totalPages = totalRowsNum / self.numPerPage;
        else:
            totalPages = (totalRowsNum / self.numPerPage) + 1 
        return totalPages
    
    def __calLastIndex(self, totalRows, totalPages,currentPageIndex):
        '''计算结束时候的索引'''
        lastIndex = 0;
        if totalRows < self.numPerPage:
            lastIndex = totalRows;
        elif ((totalRows % self.numPerPage == 0)
                or (totalRows % self.numPerPage != 0 and currentPageIndex < totalPages)) :
            lastIndex = currentPageIndex * self.numPerPage
        elif (totalRows % self.numPerPage != 0 and currentPageIndex == totalPages): # 最后一页
            lastIndex = totalRows         
        return lastIndex
```