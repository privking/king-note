# Swagger

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        List<Parameter> parameters = new ArrayList<>();
        parameters.add(new ParameterBuilder()
                .name("Authorization")
                .description("认证token")
                .modelRef(new ModelRef("string"))
                .parameterType("header")
                .required(false)
                .build());

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("xxx.controller")).paths(PathSelectors.any())
                .build()
                .globalOperationParameters(parameters);
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title(" app")
                .version("1.0")
                .description("API 描述")
                .build();
    }
}
```

```java
@ApiOperation("接口名称")
@ApiImplicitParams({
        @ApiImplicitParam(name = "fillId", value = "中文解释", dataType = "Long", required = true),
        @ApiImplicitParam(name = "year", value = "年份", dataType = "Integer", required = false)
})
@GetMapping("/queryInfo")
public Response<Object> queryInfo(Long fillId,@RequestParam(required = false) Integer year) {
    if (fillId == null) {
        return Response.failed("参数错误");
    }
    return Response.success(service.queryInfo(fillId, year));
}

@ApiOperation("接口名称")
@ApiImplicitParam(name = "name", value = "参数名称", dataType = "string", required = true)
@GetMapping("/queryByName")
public Response<List<XXXX>> queryByName(@RequestParam("name") String name) {
    return Response.success(service.queryByName(name));
}
```

```java
@Data   //lo
public class Domain {

    @ApiModelProperty("主键")   //swagger
    private Long id;
    
    @ApiModelProperty("年份")
    private Integer yearNum;

    @ApiModelProperty("月份")
    private Integer monthNum;

    @ApiModelProperty("日")
    private Integer dayNum;

    @ApiModelProperty("创建时间")
    private Date createDate;

    @ApiModelProperty("更新时间")
    private Date updateDate;
}
```
