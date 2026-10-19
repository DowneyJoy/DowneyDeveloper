Mybatis XML文件示例



```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.fancyexam.mappers.SysRoleMapper">
    <!--实体映射-->
    <resultMap id="base_result_map" type="com.fancyexam.entity.po.SysRole">
        <!--角色ID-->
        <id column="role_id" property="roleId"/>
        <!--角色名称-->
        <result column="role_name" property="roleName"/>
        <!--角色描述-->
        <result column="role_desc" property="roleDesc"/>
        <!--创建时间-->
        <result column="create_time" property="createTime"/>
        <!--最后更新时间-->
        <result column="last_update_time" property="lastUpdateTime"/>
    </resultMap>

    <!--通用查詢結果列-->
    <sql id="base_column_list">
        role_id
        ,role_name,role_desc,create_time,last_update_time
    </sql>

    <!--基礎查詢條件-->
    <sql id="base_query_condition">
        <if test="query.roleId != null">
            and role_id = #{query.roleId}
        </if>
        <if test="query.roleName != null and query.roleName!=''">
            and role_name = #{query.roleName}
        </if>
        <if test="query.roleDesc != null and query.roleDesc!=''">
            and role_desc = #{query.roleDesc}
        </if>
        <if test="query.createTime != null">
            and create_time = #{query.createTime}
        </if>
        <if test="query.lastUpdateTime != null">
            and last_update_time = #{query.lastUpdateTime}
        </if>
    </sql>

    <!--扩展查詢條件-->
    <sql id="base_query_condition_extend">
        <if test="query.roleNameFuzzy != null and query.roleNameFuzzy != ''">
            and role_name like concat('%', #{query.roleNameFuzzy}, '%')
        </if>
        <if test="query.roleDescFuzzy != null and query.roleDescFuzzy != ''">
            and role_desc like concat('%', #{query.roleDescFuzzy}, '%')
        </if>
        <if test="query.createTimeStart != null and query.createTimeStart != ''">
            <![CDATA[ and create_time >= str_to_date(#{createTimeStart}, '%Y-%m-%d') ]]>
        </if>
        <if test="query.createTimeEnd != null and query.createTimeEnd != ''">
            <![CDATA[ and create_time < date_sub(str_to_date(#{query.createTimeEnd},'%Y-%m-%d'),interval -1 day) ]]>
        </if>
        <if test="query.lastUpdateTimeStart != null and query.lastUpdateTimeStart != ''">
            <![CDATA[ and last_update_time >= str_to_date(#{lastUpdateTimeStart}, '%Y-%m-%d') ]]>
        </if>
        <if test="query.lastUpdateTimeEnd != null and query.lastUpdateTimeEnd != ''">
            <![CDATA[ and last_update_time < date_sub(str_to_date(#{query.lastUpdateTimeEnd},'%Y-%m-%d'),interval -1 day) ]]>
        </if>
    </sql>

    <!--通用查詢條件-->
    <sql id="query_condition">
        <where>
            <include refid="base_query_condition"/>
            <include refid="base_query_condition_extend"/>
        </where>
    </sql>

    <!--查询列表-->
    <select id="selectList" resultMap="base_result_map">
        SELECT
        <include refid="base_column_list"/>
        FROM sys_role
        <include refid="query_condition"/>
        <if test="query.orderBy!=null">
            order by ${query.orderBy}
        </if>
        <if test="query.simplePage!=null">
            limit #{query.simplePage.start},#{query.simplePage.end}
        </if>
    </select>

    <!--查询数量-->
    <select id="selectCount" resultType="java.lang.Integer">
        SELECT count(1) FROM sys_role
        <include refid="query_condition"/>
    </select>

    <!--插入（匹配有值的字段）-->
    <insert id="insert" parameterType="com.fancyexam.entity.po.SysRole">
        <selectKey keyProperty="bean.roleId" resultType="Integer" order="AFTER">
            SELECT LAST_INSERT_ID()
        </selectKey>
        INSERT INTO sys_role
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="bean.roleId != null">
                role_id,
            </if>
            <if test="bean.roleName != null">
                role_name,
            </if>
            <if test="bean.roleDesc != null">
                role_desc,
            </if>
            <if test="bean.createTime != null">
                create_time,
            </if>
            <if test="bean.lastUpdateTime != null">
                last_update_time,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="bean.roleId != null">
                #{bean.roleId},
            </if>
            <if test="bean.roleName != null">
                #{bean.roleName},
            </if>
            <if test="bean.roleDesc != null">
                #{bean.roleDesc},
            </if>
            <if test="bean.createTime != null">
                #{bean.createTime},
            </if>
            <if test="bean.lastUpdateTime != null">
                #{bean.lastUpdateTime},
            </if>
        </trim>
    </insert>

    <!--插入或者更新（匹配有值的字段）-->
    <insert id="insertOrUpdate" parameterType="com.fancyexam.entity.po.SysRole">
        INSERT INTO sys_role
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="bean.roleId != null">
                role_id,
            </if>
            <if test="bean.roleName != null">
                role_name,
            </if>
            <if test="bean.roleDesc != null">
                role_desc,
            </if>
            <if test="bean.createTime != null">
                create_time,
            </if>
            <if test="bean.lastUpdateTime != null">
                last_update_time,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="bean.roleId != null">
                #{bean.roleId},
            </if>
            <if test="bean.roleName != null">
                #{bean.roleName},
            </if>
            <if test="bean.roleDesc != null">
                #{bean.roleDesc},
            </if>
            <if test="bean.createTime != null">
                #{bean.createTime},
            </if>
            <if test="bean.lastUpdateTime != null">
                #{bean.lastUpdateTime},
            </if>
        </trim>
        on DUPLICATE key update
        <trim prefix="" suffix="" suffixOverrides=",">
            <if test="bean.roleName != null">
                role_name = VALUES(role_name),
            </if>
            <if test="bean.roleDesc != null">
                role_desc = VALUES(role_desc),
            </if>
            <if test="bean.createTime != null">
                create_time = VALUES(create_time),
            </if>
            <if test="bean.lastUpdateTime != null">
                last_update_time = VALUES(last_update_time),
            </if>
        </trim>
    </insert>

    <!--添加（批量添加）-->
    <insert id="insertBatch" parameterType="com.fancyexam.entity.po.SysRole">
        INSERT INTO sys_role(role_name,role_desc,create_time,last_update_time)values
        <foreach collection="list" item="item" separator=",">
            (#{item.roleName},#{item.roleDesc},#{item.createTime},#{item.lastUpdateTime})
        </foreach>
    </insert>

    <!--批量插入或更新-->
    <insert id="insertOrUpdateBatch" parameterType="com.fancyexam.entity.po.SysRole">
        INSERT INTO sys_role(role_name,role_desc,create_time,last_update_time)values
        <foreach collection="list" item="item" separator=",">
            (#{item.roleName},#{item.roleDesc},#{item.createTime},#{item.lastUpdateTime})
        </foreach>
        on DUPLICATE key update role_name = VALUES(role_name),role_desc = VALUES(role_desc),create_time =
        VALUES(create_time),last_update_time = VALUES(last_update_time)
    </insert>

    <!--根据param删除-->
    <delete id="deleteByParam">
        delete from sys_role
        <include refid="query_condition">
        </include>
    </delete>

    <!--根据param删除-->
    <update id="updateByParam" parameterType="com.fancyexam.entity.query.SysRoleQuery">
        UPDATE sys_role
        <set>
            <if test="bean.roleName != null">
                role_name = #{bean.roleName},
            </if>
            <if test="bean.roleDesc != null">
                role_desc = #{bean.roleDesc},
            </if>
            <if test="bean.createTime != null">
                create_time = #{bean.createTime},
            </if>
            <if test="bean.lastUpdateTime != null">
                last_update_time = #{bean.lastUpdateTime},
            </if>
        </set>
        <include refid="query_condition">
        </include>
    </update>

    <!-- 根据RoleId查询-->
    <select id="selectByRoleId" resultMap="base_result_map">
        select
        <include refid="base_column_list"/>
        from sys_role where role_id=#{roleId}
    </select>

    <!-- 根据RoleId更新-->
    <update id="updateByRoleId" parameterType="com.fancyexam.entity.po.SysRole">
        update sys_role
        <set>
            <if test="bean.roleId != null">
                role_id = #{bean.roleId},
            </if>
            <if test="bean.roleName != null">
                role_name = #{bean.roleName},
            </if>
            <if test="bean.roleDesc != null">
                role_desc = #{bean.roleDesc},
            </if>
            <if test="bean.createTime != null">
                create_time = #{bean.createTime},
            </if>
            <if test="bean.lastUpdateTime != null">
                last_update_time = #{bean.lastUpdateTime},
            </if>
        </set>
        where role_id=#{roleId}
    </update>

    <!-- 根据RoleId删除-->
    <delete id="deleteByRoleId">
        delete
        from sys_role
        where role_id = #{roleId}
    </delete>
</mapper>

```

