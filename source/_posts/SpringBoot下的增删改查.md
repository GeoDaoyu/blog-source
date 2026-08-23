---
title: SpringBoot下的增删改查
date: 2020-07-03 20:22:42
tags:
- Java
- Spring Boot
categories:
- Java
---
实习的时候，需要临时写点后端代码，实现最基础的增删改查。

在已有的项目框架（Spring Boot）中，观察其他模块的实现，发现需要7个文件。

热心的同事还专门电话给我讲解spring的目录结构和层级关系。

整理记录一下，下次需要写的时候，就直接凑7文件了【狗头】。
<!-- more -->
## 目录结构

```
src/main
├──  java/com.geodaoyu
|    ├──  manager
|    |    ├──  data/model/MsgDialog.java
|    |    ├──  mapper/MsgDialogMapper.java
|    |    ├──  filter/MsgDialogFilter.java
|    |    └──  service
|    |         ├──  impls/MsgDialogServiceImpl.java
|    |         └──  MsgDialogService.java
|    └──  webcontroller/PortalManagerController.java
└──  resources/mapper
     └──  MsgDialogMapper.xml
```

## 层级关系

+ model层，数据库实体层，定义实体
+ dao层，数据持久层，访问数据库，向数据库发送sql语句
+ service层，业务逻辑层
+ controller层，控制层

## model

相对路径：`com/geodaoyu/manager/data/model/MsgDialog.java`

```java
package com.geodaoyu.manager.data.model;

/**
 * @descript: 消息提醒Model类
 * @author: zhanggy
 * @Date: 2018/08/21
 */
public class MsgDialog {
    private String msgId; //消息唯一标识标识
    private String sender; //发送人
    private String receiver; //接收人
    private String sendTime; //发送时间
    private String content; //发送内容
    private String state;  //消息状态
    private String readTime; //查看时间

    public String getMsgId() {
        return msgId;
    }

    public void setMsgId(String msgId) {
        this.msgId = msgId;
    }

    public String getSender() {
        return sender;
    }

    ...
}
```

> IntelliJ IDEA 可以使用快捷键`ALT`+`INSERT`生成get和set。

## filter

相对路径：`com/geodaoyu/manager/filter/MsgDialogFilter.java`

```java
package com.geodaoyu.manager.filter;

/**
 * @descript: 消息提醒过滤类
 * @author: zhanggy
 * @Date: 2018/08/21
 */
public class MsgDialogFilter {
    private String msgId; //消息唯一标识标识
    private String receiver; //接收人
    private String sendTime; //发送时间
    private String content; //发送内容
    private String state;  //消息状态
    private String readTime; //查看时间
    private int startIndex; //起始索引
    private int endIndex; //终止索引

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    ...
}
```

## model mapper

相对路径：`com/geodaoyu/manager/data/mapper/MsgDialogMapper.java`

```java
package com.geodaoyu.manager.data.mapper;

import com.geodaoyu.manager.data.model.MsgDialog;
import com.geodaoyu.manager.filter.MsgDialogFilter;

import java.util.List;

public interface MsgDialogMapper {
    int insertMsg(MsgDialog msgDialog);
    int insertMsgs(List<MsgDialog> msgDialogs);
    int deleteMsg(MsgDialog msgDialog);
    int updateMsg(MsgDialog msgDialog);
    int updateMsgs(MsgDialog msgDialog);
    List<MsgDialog> selectMsgsByFilter(MsgDialogFilter filter);
    int selectMsgsCountByFilter(MsgDialogFilter filter);
}
```

## service

相对路径：`com/geodaoyu/manager/service/MsgDialogService.java`

```java
package com.geodaoyu.manager.service;

import com.geodaoyu.manager.data.model.MsgDialog;
import com.geodaoyu.manager.filter.MsgDialogFilter;

import java.util.List;

public interface MsgDialogService {
    int insertMsg(MsgDialog msgDialog);
    int insertMsgs(List<MsgDialog> msgDialogs);
    int deleteMsg(MsgDialog msgDialog);
    int updateMsg(MsgDialog msgDialog);
    int updateMsgs(MsgDialog msgDialog);
    List<MsgDialog> selectMsgsByFilter(MsgDialogFilter filter);
    int selectMsgsCountByFilter(MsgDialogFilter filter);
}
```

## impl

相对路径：`com/geodaoyu/manager/service/impls/MsgDialogServiceImpl.java`

```java
package com.geodaoyu.manager.service.impls;

import com.geodaoyu.manager.data.mapper.MsgDialogMapper;
...

/**
 * @descript:
 * @author: zhanggy
 * @Date: 2018/08/21
 */
@Service(value = "MsgDialogService")
public class MsgDialogServiceImpl implements MsgDialogService {

    @Autowired
    private MsgDialogMapper msgDialogMapper;

    /*
    插入单条消息
     */
    @Override
    public int insertMsg(MsgDialog msgDialog) {
        msgDialog.setMsgId(UUID.randomUUID().toString());
        MsgHandler msgHandler = new MsgHandler();
        msgHandler.sendMsg(msgDialog.getReceiver(), new TextMessage("message"));
        return msgDialogMapper.insertMsg(msgDialog);
    }

    /*
    批量插入消息
     */
    @Override
    public int insertMsgs(List<MsgDialog> msgDialogs) {
        MsgDialog msgDialog =  msgDialogs.get(0);
        MsgHandler msgHandler = new MsgHandler();
        msgHandler.sendMsg(msgDialog.getReceiver(), new TextMessage("message"));
        return msgDialogMapper.insertMsgs(msgDialogs);
    }

    /*
    根据MSGID删除消息
     */
    @Override
    public int deleteMsg(MsgDialog msgDialog) {
        return msgDialogMapper.deleteMsg(msgDialog);
    }

    /*
    更改消息状态
     */
    @Override
    public int updateMsg(MsgDialog msgDialog) {
        msgDialog.setState("已读");
        msgDialog.setReadTime(NowTime.GetTimeString2());
        return msgDialogMapper.updateMsg(msgDialog);
    }

    /*
    批量将所有未读更改为已读
     */
    @Override
    public int updateMsgs(MsgDialog msgDialog) {
        msgDialog.setState("已读");
        msgDialog.setReadTime(NowTime.GetTimeString2());
        return msgDialogMapper.updateMsgs(msgDialog);
    }

    /*
    根据接收者查询消息，按时间倒序排列
     */
    @Override
    public List<MsgDialog> selectMsgsByFilter(MsgDialogFilter filter) {
        return msgDialogMapper.selectMsgsByFilter(filter);
    }

    /*
    查询消息数量
     */
    @Override
    public int selectMsgsCountByFilter(MsgDialogFilter filter) {
        return msgDialogMapper.selectMsgsCountByFilter(filter);
    }
}
```

## dao

相对路径：`mapper/MsgDialogMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.geodaoyu.manager.data.mapper.MsgDialogMapper">
    <resultMap id="BaseResultMap" type="com.geodaoyu.manager.data.model.MsgDialog">
        <id property="msgId" column="MSGID"  javaType="java.lang.String" jdbcType="VARCHAR"/>
        <result property="sender" column="SENDER" javaType="java.lang.String" jdbcType="VARCHAR"/>
        <result property="receiver" column="RECEIVER" javaType="java.lang.String" jdbcType="VARCHAR"/>
        <result property="sendTime" column="SENDTIME" javaType="java.lang.String" jdbcType="VARCHAR"/>
        <result property="content" column="CONTENT" javaType="java.lang.String" jdbcType="VARCHAR"/>
        <result property="state" column="STATE" javaType="java.lang.String" jdbcType="VARCHAR"/>
        <result property="readTime" column="READTIME" javaType="java.lang.String" jdbcType="VARCHAR"/>
    </resultMap>

    <insert id="insertMsg" parameterType="com.geodaoyu.manager.data.model.MsgDialog">
        INSERT INTO
        w_message
        (
          MSGID,SENDER,RECEIVER,SENDTIME,CONTENT,STATE,READTIME
        )
        VALUES
        (
        #{msgId,jdbcType=VARCHAR},
        #{sender,jdbcType=VARCHAR},
        #{receiver,jdbcType=VARCHAR},
        #{sendTime,jdbcType=VARCHAR},
        #{content,jdbcType=VARCHAR},
        #{state,jdbcType=VARCHAR},
        #{readTime,jdbcType=VARCHAR}
        )
    </insert>

    <insert id="insertMsgs" parameterType="java.util.List" >
        INSERT ALL
        <foreach collection="list" item="item" separator="  ">
            INTO
            w_message
            (
            MSGID,SENDER,RECEIVER,SENDTIME,CONTENT,STATE,READTIME
            )
            VALUES
            (
            #{item.msgId,jdbcType=VARCHAR},
            #{item.sender,jdbcType=VARCHAR},
            #{item.receiver,jdbcType=VARCHAR},
            #{item.sendTime,jdbcType=VARCHAR},
            #{item.content,jdbcType=VARCHAR},
            #{item.state,jdbcType=VARCHAR},
            #{item.readTime,jdbcType=VARCHAR},
            )
        </foreach>
        select * from dual
    </insert>

    <delete id="deleteMsg" parameterType="com.geodaoyu.manager.data.model.MsgDialog">
        DELETE FROM
          w_message
        WHERE
          MSGID=#{msgId,jdbcType=VARCHAR}
     </delete>

    <update id="updateMsg" parameterType="com.geodaoyu.manager.data.model.MsgDialog">
        UPDATE
          w_message
        SET
          STATE=#{state,jdbcType=VARCHAR},
          READTIME=#{readTime,jdbcType=VARCHAR}
        WHERE
          MSGID=#{msgId,jdbcType=VARCHAR}
    </update>

    <update id="updateMsgs" parameterType="com.geodaoyu.manager.data.model.MsgDialog">
    UPDATE
      w_message
    SET
      STATE='已读',
      READTIME=#{readTime,jdbcType=VARCHAR}
    WHERE
      RECEIVER=#{receiver,jdbcType=VARCHAR}
      AND
      STATE='未读'
    </update>

    <select id="selectMsgsByFilter" parameterType="com.geodaoyu.manager.filter.MsgDialogFilter" resultMap="BaseResultMap">
        SELECT * FROM
        (
        SELECT r.*,ROWNUM oracleRN FROM
        (
        SELECT t.* FROM w_message t
        <trim prefix="WHERE" prefixOverrides="AND">
            <if test="receiver != null and receiver != ''">
                AND t.RECEIVER = #{receiver,jdbcType=VARCHAR}
            </if>
            <if test="state != null and state != ''">
                AND t.STATE  = #{state,jdbcType=VARCHAR}
            </if>
            <if test="content != null and content != ''">
                AND t.CONTENT LIKE CONCAT('%',CONCAT(#{content},'%'))
            </if>
        </trim>
        ORDER BY STATE DESC, SENDTIME DESC
        ) r
        )
        <trim prefix="WHERE" prefixOverrides="AND">
            <if test="startIndex != null and startIndex &gt; 0" >
                AND oracleRN &gt;= ${startIndex}
            </if>
            <if test="endIndex != null and endIndex &gt; 0" >
                AND oracleRN &lt;= ${endIndex}
            </if>
        </trim>
    </select>

    <select id="selectMsgsCountByFilter" parameterType="com.geodaoyu.manager.filter.MsgDialogFilter" resultType="java.lang.Integer">
        SELECT
        COUNT(*)
        FROM
        w_message t
        <trim prefix="WHERE" prefixOverrides="AND">
            <if test="receiver != null and receiver != ''">
                AND t.RECEIVER = #{receiver,jdbcType=VARCHAR}
            </if>
            <if test="state != null and state != ''">
                AND t.STATE  = #{state,jdbcType=VARCHAR}
            </if>
            <if test="content != null and content != ''">
                AND t.CONTENT LIKE CONCAT('%',CONCAT(#{content},'%'))
            </if>
        </trim>
    </select>
</mapper>
```

> 注意：oracle和pg的分页查询写法不同

## controller

相对路径：`com/geodaoyu/webcontroller/MsgDialogController.java`

```java
package com.geodaoyu.webcontroller;

import com.geodaoyu.manager.data.model.MsgDialog;
...

@Api(value="/msgDialog", description = "消息接口")
@Controller
@RequestMapping(value="/msgDialog")
@CrossOrigin
public class MsgDialogController {

    @Autowired
    private MsgDialogService msgDialogService;

    @Autowired
    private LogService logService;

    @RequestMapping(value="/insertMsg", method = RequestMethod.POST, produces = "application/json; charset=utf-8")
    @ResponseBody
    @ApiOperation(value = "添加消息",notes = "添加消息")
    @ApiImplicitParam(name = "msgDialog", value = "MsgDialog 模型", required = true)
    public Map insertMsgDialog(@RequestBody MsgDialog msgDialog) {
        Map map = new HashMap();
        int insertMsg = msgDialogService.insertMsg(msgDialog);
        map.put("insert", insertMsg);
        return map;
    }

    @RequestMapping(value="/deleteMsg", method = RequestMethod.POST, produces = "application/json; charset=utf-8")
    @ResponseBody
    @ApiOperation(value = "删除消息",notes = "删除消息")
    @ApiImplicitParam(name = "msgDialog", value = "MsgDialog 模型", required = true)
    public Map deleteMsgDialog(@RequestBody MsgDialog msgDialog) {
        Map map = new HashMap();
        int deleteMsg = msgDialogService.deleteMsg(msgDialog);
        map.put("delete", deleteMsg);
        return map;
    }

    @RequestMapping(value="/updateMsg", method = RequestMethod.POST, produces = "application/json; charset=utf-8")
    @ResponseBody
    @ApiOperation(value = "更改消息状态",notes = "更改消息状态")
    @ApiImplicitParam(name = "msgDialog", value = "MsgDialog 模型", required = true)
    public Map updateMsgDialog(@RequestBody MsgDialog msgDialog) {
        Map map = new HashMap();
        int updateMsg = msgDialogService.updateMsg(msgDialog);
        map.put("update", updateMsg);
        return map;
    }

    @RequestMapping(value="/updateMsgs", method = RequestMethod.POST, produces = "application/json; charset=utf-8")
    @ResponseBody
    @ApiOperation(value = "批量更改消息状态",notes = "批量更改消息状态")
    @ApiImplicitParam(name = "msgDialog", value = "MsgDialog 模型", required = true)
    public Map updateMsgDialogs(@RequestBody MsgDialog msgDialog) {
        Map map = new HashMap();
        int updateMsgs = msgDialogService.updateMsgs(msgDialog);
        map.put("update", updateMsgs);
        return map;
    }

    @RequestMapping(value="/select", method = RequestMethod.POST, produces = "application/json; charset=utf-8")
    @ResponseBody
    @ApiOperation(value = "按条件查询消息",notes = "按条件查询消息")
    @ApiImplicitParam(name = "msgDialogFilter", value = "msgDialogFilter 模型", required = true)
    public Map selectMsgsByFilter(@RequestBody MsgDialogFilter msgDialogFilter) {
        Map map = new HashMap();
        List<MsgDialog> pageData = msgDialogService.selectMsgsByFilter(msgDialogFilter);
        map.put("pageData", pageData);
        return map;
    }

    @RequestMapping(value="/selectCount", method = RequestMethod.POST, produces = "application/json; charset=utf-8")
    @ResponseBody
    @ApiOperation(value = "按条件查询消息数量",notes = "按条件查询消息数量")
    @ApiImplicitParam(name = "msgDialogFilter", value = "msgDialogFilter 模型", required = true)
    public Map selectMsgsCountByFilter(@RequestBody MsgDialogFilter msgDialogFilter) {
        Map map = new HashMap();
        int count = msgDialogService.selectMsgsCountByFilter(msgDialogFilter);
        map.put("count", count);
        return map;
    }
}
```

