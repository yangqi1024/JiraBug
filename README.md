# 统计JiraBug

### 需要安装python 并且安装jira
~~~
pip install jira
~~~

使用python统计jira上的bug

~~~ python
import datetime

from jira import JIRA

# jira 登录名
name = ''
# jira 登录密码
password = ''

# 这里可以加入其他人的名字
users = [name]
# 这个是中文对称表
chineseName = {name: 'xxx'}
# jira地址 类似于https://host:port/jira/
jiraUrl = ''

bugs = {}

first = datetime.date(datetime.date.today().year, datetime.date.today().month - 1, 1)
last = datetime.date(datetime.date.today().year, datetime.date.today().month, 1) - datetime.timedelta(1)

jira = JIRA(jiraUrl, basic_auth=(name, password))

def querySolvedBug(user):
    jql = 'issuetype = Bug AND created >= %s AND created <= %s AND 解决人 in (%s) order by updated DESC' % (
        first, last, user)
    reopenJql = 'issuetype = Bug AND status = Reopened AND created >= %s AND created <= %s AND 解决人 in (%s) order by updated DESC' % (
        first, last, user)

    content = jira.search_issues(jql, json_result=True)

    issues = content['issues']
    for issue in issues:
        bugs[user]['bugs'].append({"key": issue['key'], "status": issue['fields']['status']['name'],
                                   "level": issue['fields']['customfield_10302']['value']})

    content = jira.search_issues(reopenJql, json_result=True)
    issues = content['issues']
    for issue in issues:
        bugs[user]['reopen'].append({"key": issue['key'], "status": issue['fields']['status']['name'],
                                     "level": issue['fields']['customfield_10302']['value']})

    print()


#     查询经办人挂着的bug
def queryssigneeBug(user):
    jql = 'issuetype = Bug AND created >= %s AND created <= %s AND assignee in (%s) order by updated DESC' % (
        first, last, user)
    reopenJql = 'issuetype = Bug AND status = Reopened AND created >= %s AND created <= %s AND assignee in (%s) order by updated DESC' % (
        first, last, user)
    content = jira.search_issues(jql, json_result=True)
    issues = content['issues']
    for issue in issues:
        bugs[user]['bugs'].append({"key": issue['key'], "status": issue['fields']['status']['name'],
                                   "level": issue['fields']['customfield_10302']['value']})

    content = jira.search_issues(reopenJql, json_result=True)
    issues = content['issues']
    for issue in issues:
        bugs[user]['reopen'].append({"key": issue['key'], "status": issue['fields']['status']['name'],
                                     "level": issue['fields']['customfield_10302']['value']})

    print()


def printInfo(user):
    print("%s" % (chineseName[user]))
    print("---------------------------------------------------")
    print('%s 至 %s , Bug数为：%s' % (first, last, len(bugs[user]['bugs'])))
    for bug in bugs[user]['bugs']:
        print("Bug号：%s,\t状态：%s,\t等级：%s" % (
            bug['key'], bug['status'], bug['level']))
    print("---------------------------------------------------")
    print('%s 至 %s , 重复激活Bug数为：%s' % (first, last, len(bugs[user]['reopen'])))
    for bug in bugs[user]['reopen']:
        print("Bug号：%s,\t状态：%s,\t等级：%s" % (
            bug['key'], bug['status'], bug['level']))


for user in users:
    bugs[user] = {'bugs': [], 'reopen': []}
    querySolvedBug(user)
    queryssigneeBug(user)
    printInfo(user)

~~~
