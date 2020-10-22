## 1. foreach循环
## 1.1查询
```
 List<MapPostedInfo> selectListByStatusList(@Param("statusList") List<String> statusList);

//查询不到数据的情况下，如果返回结果类型是对象，则值为null，如果返回结果类型是List，则值为size=0的list
<select id="selectListByStatusList" resultMap="BaseResultMap" >
		SELECT
		<include refid="Base_Column_List" />
		FROM table
		WHERE  is_deleted = 0 and status in
			<foreach item="item" index="index" collection="statusList" open="(" separator="," close=")">
				#{item}
			</foreach>
	</select>
```

## 1.2插入
```
<insert id ="insertBatch" parameterType="java.util.List" >
        insert into intelligent_cabinet_config
        (device_code, type)
        values
        <foreach collection ="intelligentCabinetConfigList" item="item" separator=",">
            (
            #{item.deviceCode,jdbcType=VARCHAR},
            #{item.type,jdbcType=VARCHAR}
            )
        </foreach >
    </insert >
```