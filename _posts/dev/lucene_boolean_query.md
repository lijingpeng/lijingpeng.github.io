title: lucene 布尔查询的使用
date: 2015-11-26 20:53:56
tags: lucene
category: dev
---

各种查询
方式一：使用QueryParser与查询语法。（会使用分词器）

MultiFieldQueryParser
查询字符串 ------------------------> Query对象

例如：
上海 AND 天气
上海 OR 天气
上海新闻 AND site:news.163.com
...

方式二：
直接创建Query的实例（子类的），不会使用分词器
new TermQuery(..);
new BooleanQuery(..);

```java
package cn.itcast.i_query;

import java.util.ArrayList;
import java.util.List;

import org.apache.lucene.document.Document;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.BooleanQuery;
import org.apache.lucene.search.FuzzyQuery;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.NumericRangeQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.TermQuery;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.search.WildcardQuery;
import org.apache.lucene.search.BooleanClause.Occur;
import org.junit.Test;

import cn.itcast._domain.Article;
import cn.itcast._util.ArticleDocumentUtils;
import cn.itcast._util.LuceneUtils;

public class TestApp {

    // 关键词查询
    @Test
    public void testTermQuery() {
        // 对应的查询字符串为：title:lucene
        TermQuery query = new TermQuery(new Term("title", "lucene"));
        searchAndShowResult(query);
    }

    // 通配符查询
    // ? 表示一个任意字符
    // * 表示0或多个任意字符
    @Test
    public void testWildcardQuery() {
        // 对应的查询字符串为：title:lu*n?
        // WildcardQuery query = new WildcardQuery(new Term("title", "lu*n?"));

        // 对应的查询字符串为：content:互?网
        WildcardQuery query = new WildcardQuery(new Term("content", "互?网"));
        searchAndShowResult(query);
    }

    // 查询所有
    @Test
    public void testMatchAllDocsQuery() {
        // 对应的查询字符串为：*:*
        MatchAllDocsQuery query = new MatchAllDocsQuery();
        searchAndShowResult(query);
    }

    // 模糊查询
    @Test
    public void testFuzzyQuery() {
        // 对应的查询字符串为：title:lucenX~0.9
        // 第二个参数是最小相似度，表示有多少正确的就显示出来，比如0.9表示有90%正确的字符就会显示出来。
        FuzzyQuery query = new FuzzyQuery(new Term("title", "lucenX"), 0.8F);
        searchAndShowResult(query);
    }

    // 范围查询
    @Test
    public void testNumericRangeQuery() {
        // 对应的查询字符串为：id:[5 TO 15]
        // NumericRangeQuery query = NumericRangeQuery.newIntRange("id", 5, 15, true, true);

        // 对应的查询字符串为：id:{5 TO 15}
        // NumericRangeQuery query = NumericRangeQuery.newIntRange("id", 5, 15, false, false);

        // 对应的查询字符串为：id:[5 TO 15}
        NumericRangeQuery query = NumericRangeQuery.newIntRange("id", 5, 15, true, false);

        searchAndShowResult(query);
    }

    // 布尔查询
    @Test
    public void testBooleanQuery() {
        BooleanQuery booleanQuery = new BooleanQuery();
        // booleanQuery.add(query, Occur.MUST); // 必须满足
        // booleanQuery.add(query, Occur.SHOULD); // 多个SHOULD一起用表示OR的关系
        // booleanQuery.add(query, Occur.MUST_NOT); // 非

        Query query1 = new TermQuery(new Term("title", "lucene"));
        Query query2 = NumericRangeQuery.newIntRange("id", 5, 15, false, true);

        // // 对应的查询字符串为：+title:lucene +id:{5 TO 15]
        // // 对应的查询字符串为（大写的AND）：title:lucene AND id:{5 TO 15]
        // booleanQuery.add(query1, Occur.MUST);
        // booleanQuery.add(query2, Occur.MUST);

        // 对应的查询字符串为：title:lucene id:{5 TO 15]
        // 对应的查询字符串为：title:lucene OR id:{5 TO 15]
        // booleanQuery.add(query1, Occur.SHOULD);
        // booleanQuery.add(query2, Occur.SHOULD);

        // 对应的查询字符串为：+title:lucene -id:{5 TO 15]
        // 对应的查询字符串为：title:lucene (NOT id:{5 TO 15] )
        booleanQuery.add(query1, Occur.MUST);
        booleanQuery.add(query2, Occur.MUST_NOT);

        searchAndShowResult(booleanQuery);
    }

    /**
     * 测试搜索的工具方法
     * 
     * @param query
     */
    private void searchAndShowResult(Query query) {
        try {
            // // 准备查询条件
            // String queryString = "content:lucene";
            // // 1，把查询字符串转为Query对象（从title和content中查询）
            // QueryParser queryParser = new MultiFieldQueryParser(Version.LUCENE_30, new String[] { "title", "content" }, LuceneUtils.getAnalyzer());
            // Query query = queryParser.parse(queryString);

            System.out.println("--->  // 对应的查询字符串为：" + query + "\n");

            // 2，执行查询，得到中间结果
            IndexSearcher indexSearcher = new IndexSearcher(LuceneUtils.getDirectory()); // 指定所用的索引库
            TopDocs topDocs = indexSearcher.search(query, 100); // 最多返回前n条结果

            // 3，处理结果
            List<Article> list = new ArrayList<Article>();
            for (int i = 0; i < topDocs.scoreDocs.length; i++) {
                // 根据编号拿到Document数据
                int docId = topDocs.scoreDocs[i].doc; // Document的内部编号
                Document doc = indexSearcher.doc(docId);
                // 把Document转为Article
                Article article = ArticleDocumentUtils.documentToArticle(doc);
                list.add(article);
            }
            indexSearcher.close();

            // 显示结果
            System.out.println("总结果数：" + list.size());
            for (Article a : list) {
                System.out.println("------------------------------");
                System.out.println("id = " + a.getId());
                System.out.println("title = " + a.getTitle());
                System.out.println("content = " + a.getContent());
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

From: http://www.cnblogs.com/friends-wf/p/3796721.html
