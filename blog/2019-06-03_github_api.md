# Github Api 

## 认证机制 [链接](<https://developer.github.com/v3/#authentication>)

### 认证方式

1. 基础认证方式 `curl -u "username" https://api.github.com`或者`curl -u username:token https://api.github.com/user`
2. OAuth认证方式`curl -H "Authorization: token OAUTH-TOKEN" https://api.github.com`
3. Client ID 和 Client Secret认证``

### 认证TOKEN及Client ID

TOKEN：进入github账号`settings` -> `Personal access tokens` -> `Generate new token`生成token，开启repo的权限

Client ID：进入github账号`settings` -> `OAuth Apps` 

## 频率限制

1. 对于core、search、graphql、integration_mainfest有自定义的频率限制，其他的是统一的频率限制，可通过`/rate_limit`接口查看频率限制，返回限制数量、剩余数量、刷新时间等数据。[链接](<https://developer.github.com/v3/rate_limit/>)
2. 每次请求，响应头中都会返回`X-RateLimit`、`X-RateLimit-Remaining`、`X-RateLimit-Reset`参数，更新限制数量、剩余数量、刷新时间的
3. 在search中，对于basic authentication、OAuth、或者client_id和secret的，每分钟最多可以有30个请求。未经验证的请求，每分钟最多10个请求。[链接](<https://developer.github.com/v3/search/#rate-limit>)
4. 单个用户的所有令牌，每小时共享5000个请求。[链接](<https://developer.github.com/v3/#rate-limiting>)
5. 经过认证的请求会增加一定量频率限制



## 请求头要求

1. 请求头携带属性`User-Agent`[链接](<https://developer.github.com/v3/#user-agent-required>)



## API接口 [链接](<https://developer.github.com/v3/search/>)

### 1. 搜索仓库

```
GET /search/repositories
```

**参数**

|   Name   |   Type   |                         Description                          |
| :------: | :------: | :----------------------------------------------------------: |
|   `q`    | `string` | **Required**. The query contains one or more search keywords and qualifiers. Qualifiers allow you to limit your search to specific areas of GitHub. The REST API supports the same qualifiers as GitHub.com. To learn more about the format of the query, see [Constructing a search query](https://developer.github.com/v3/search/#constructing-a-search-query). See "[Searching for repositories](https://help.github.com/articles/searching-for-repositories/)" for a detailed list of qualifiers. |
|  `sort`  | `string` | Sorts the results of your query by number of `stars`, `forks`, or `help-wanted-issues` or how recently the items were `updated`. Default: [best match](https://developer.github.com/v3/search/#ranking-search-results) |
| `order`  | `string` | Determines whether the first search result returned is the highest number of matches (`desc`) or lowest number of matches (`asc`). This parameter is ignored unless you provide `sort`. Default: `desc` |
| per_page |   int    |                           max 100                            |
|   page   |   int    |                           1-based                            |

**示例**

通用的查询

```
curl https://api.github.com/search/repositories?q=tetris+language:assembly&sort=stars&order=desc
```

多主题搜索

```
curl -H "Accept: application/vnd.github.mercy-preview+json" \
https://api.github.com/search/repositories?q=topic:ruby+topic:rails
```

**限定符 **[链接](<https://help.github.com/en/articles/searching-for-repositories>)

`in:name`，例：`jquery in:name` 限定仓库名称包含jquery

`in:description`，例： `jquery in:name,description` 限定仓库名称或描述中包含jquery

`in:readme`，例：`jquery in:readme` 限定readme文件中包含jquery

`user: USERNAME`，例：`user:defunkt forks:>100` 限定用户未defunkt并且有大于100个fork

`org: ORGNAME`，例：`org:github`限定组织为github

`size: n`， 例：`size:1000`、`size:>=30000`、`size:<50`、`size:50..120` 限定仓库大小为1MB、大于30MB、小于50KB、在50KB到120KB之间

`forks: n`，例：`forks:5` 分支数量匹配

`stars: n`，例：`stars:10..20` 星数在10到20之间

`created: YYYY-MM-DD`

`pushed: YYYY-MM-DD`

`language: LANGUAGE`

`topic: TOPIC`，例：`topic:jekyll`主题分类是jekyll的

`topics: n`，例：`topics:5`有5个主题分类的

`license: LICENSE_KEYWORD`，例：`license:apache-2.0`匹配证书是Apache License 2.0的 

`is:public`或`is:private`

`mirror:true`或`mirror:false`

`archived:true`或`archived:false `

`repo:USERNAME/REPONAME`

### 2. 搜索提交记录

```
GET /search/commits
```

**参数**

|   Name   |   Type   |                         Description                          |
| :------: | :------: | :----------------------------------------------------------: |
|   `q`    | `string` | **Required**. The query contains one or more search keywords and qualifiers. Qualifiers allow you to limit your search to specific areas of GitHub. The REST API supports the same qualifiers as GitHub.com. To learn more about the format of the query, see [Constructing a search query](https://developer.github.com/v3/search/#constructing-a-search-query). See "[Searching commits](https://help.github.com/articles/searching-commits/)" for a detailed list of qualifiers. |
|  `sort`  | `string` | Sorts the results of your query by `author-date` or `committer-date`. Default: [best match](https://developer.github.com/v3/search/#ranking-search-results) |
| `order`  | `string` | Determines whether the first search result returned is the highest number of matches (`desc`) or lowest number of matches (`asc`). This parameter is ignored unless you provide `sort`. Default: `desc` |
| per_page |   int    |                           max 100                            |
|   page   |   int    |                           1-based                            |

**示例**

在octocat/Spoon-Knife仓库搜索css有关的提交

```
curl -H "Accept: application/vnd.github.cloak-preview" \
https://api.github.com/search/commits?q=repo:octocat/Spoon-Knife+css
```

### 3. 搜索代码

```
GET /search/code
```

处于搜索的复杂性，有几个搜索限制：

1. 默认仅搜索默认分支，大部分情况下`master`分支是默认分支
2. 仅搜索小于384K的文件
3. 必须包含一个关键词，`language:go`是非法的，而`amazing language:go`是可行的

**参数**

|   Name   |   Type   |                         Description                          |
| :------: | :------: | :----------------------------------------------------------: |
|   `q`    | `string` | **Required**. The query contains one or more search keywords and qualifiers. Qualifiers allow you to limit your search to specific areas of GitHub. The REST API supports the same qualifiers as GitHub.com. To learn more about the format of the query, see [Constructing a search query](https://developer.github.com/v3/search/#constructing-a-search-query). See "[Searching code](https://help.github.com/articles/searching-code/)" for a detailed list of qualifiers. |
|  `sort`  | `string` | Sorts the results of your query. Can only be `indexed`, which indicates how recently a file has been indexed by the GitHub search infrastructure. Default: [best match](https://developer.github.com/v3/search/#ranking-search-results) |
| `order`  | `string` | Determines whether the first search result returned is the highest number of matches (`desc`) or lowest number of matches (`asc`). This parameter is ignored unless you provide `sort`. Default: `desc` |
| per_page |   int    |                           max 100                            |
|   page   |   int    |                           1-based                            |

**示例**

在jQuery仓库中查找`addClass` 

```
curl https://api.github.com/search/code?q=addClass+in:file+language:js+repo:jquery/jquery
```

**接受匹配偏移**

请求头中增加`Accept: application/vnd.github.v3.text-match+json`参数，将会在响应信息中额外接收到`text-matches`字段信息，text-matches是一组列表，列表内对象包含`object_url`、`object_type`、`property`、`fragment`、`matches`，它们用来表示查询字符串在代码中的匹配位置。[链接](<https://developer.github.com/v3/search/#text-match-metadata>)

**注：在没有Authorization的情况下，存在必须指定[user|repo|org]中的一个参数限制**

### 3. 搜索问题

```
GET /search/issues
```

**参数**

|   Name   |   Type   |                         Description                          |
| :------: | :------: | :----------------------------------------------------------: |
|   `q`    | `string` | **Required**. The query contains one or more search keywords and qualifiers. Qualifiers allow you to limit your search to specific areas of GitHub. The REST API supports the same qualifiers as GitHub.com. To learn more about the format of the query, see [Constructing a search query](https://developer.github.com/v3/search/#constructing-a-search-query). See "[Searching issues and pull requests](https://help.github.com/articles/searching-issues-and-pull-requests/)" for a detailed list of qualifiers. |
|  `sort`  | `string` | Sorts the results of your query by the number of `comments`, `reactions`, `reactions-+1`, `reactions--1`, `reactions-smile`, `reactions-thinking_face`, `reactions-heart`, `reactions-tada`, or `interactions`. You can also sort results by how recently the items were `created` or `updated`, Default: [best match](https://developer.github.com/v3/search/#ranking-search-results) |
| `order`  | `string` | Determines whether the first search result returned is the highest number of matches (`desc`) or lowest number of matches (`asc`). This parameter is ignored unless you provide `sort`. Default: `desc` |
| per_page |   int    |                           max 100                            |
|   page   |   int    |                           1-based                            |

**示例**

查找最早的未解决的python问题

```
curl https://api.github.com/search/issues?q=windows+label:bug+language:python+state:open&sort=created&order=asc
```

### 4. 搜索用户

```
GET /search/users
```

**参数**

|   Name   |   Type   |                         Description                          |
| :------: | :------: | :----------------------------------------------------------: |
|   `q`    | `string` | **Required**. The query contains one or more search keywords and qualifiers. Qualifiers allow you to limit your search to specific areas of GitHub. The REST API supports the same qualifiers as GitHub.com. To learn more about the format of the query, see [Constructing a search query](https://developer.github.com/v3/search/#constructing-a-search-query). See "[Searching users](https://help.github.com/articles/searching-users/)" for a detailed list of qualifiers. |
|  `sort`  | `string` | Sorts the results of your query by number of `followers` or `repositories`, or when the person `joined` GitHub. Default: [best match](https://developer.github.com/v3/search/#ranking-search-results) |
| `order`  | `string` | Determines whether the first search result returned is the highest number of matches (`desc`) or lowest number of matches (`asc`). This parameter is ignored unless you provide `sort`. Default: `desc` |
| per_page |   int    |                           max 100                            |
|   page   |   int    |                           1-based                            |

**示例**

查询热门用户

```
curl https://api.github.com/search/users?q=tom+repos:%3E42+followers:%3E1000
```

### 5. 搜索主题

```
GET /search/topics
```

**参数**

|   Name   |   Type   |                         Description                          |
| :------: | :------: | :----------------------------------------------------------: |
|   `q`    | `string` | **Required**. The query contains one or more search keywords and qualifiers. Qualifiers allow you to limit your search to specific areas of GitHub. The REST API supports the same qualifiers as GitHub.com. To learn more about the format of the query, see [Constructing a search query](https://developer.github.com/v3/search/#constructing-a-search-query). |
| per_page |   int    |                           max 100                            |
|   page   |   int    |                           1-based                            |

**示例**

搜索精选的ruby有关的主题

```
curl -H 'Accept: application/vnd.github.mercy-preview+json' \
'https://api.github.com/search/topics?q=ruby+is:featured'
```

### 6. 搜索标签

```
GET /search/labels
```

**参数**

|      Name       |   Type    |                         Description                          |
| :-------------: | :-------: | :----------------------------------------------------------: |
| `repository_id` | `integer` |           **Required**. The id of the repository.            |
|       `q`       | `string`  | **Required**. The search keywords. This endpoint does not accept qualifiers in the query. To learn more about the format of the query, see [Constructing a search query](https://developer.github.com/v3/search/#constructing-a-search-query). |
|     `sort`      | `string`  | Sorts the results of your query by when the label was `created` or `updated`. Default: [best match](https://developer.github.com/v3/search/#ranking-search-results) |
|     `order`     | `string`  | Determines whether the first search result returned is the highest number of matches (`desc`) or lowest number of matches (`asc`). This parameter is ignored unless you provide `sort`. Default: `desc` |
|    per_page     |    int    |                           max 100                            |
|      page       |    int    |                           1-based                            |

**示例**

在指定仓库查找`bug`、`defect`、`eenhancement`有关的标签

```
curl -H 'Accept: application/vnd.github.symmetra-preview+json' \
'https://api.github.com/search/labels?repository_id=64778136&q=bug+defect+enhancement'
```

