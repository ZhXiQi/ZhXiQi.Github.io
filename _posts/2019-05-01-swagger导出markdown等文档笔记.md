---
layout: post
title: swagger导出markdown等文档笔记
data: 2019-05-01
tag: swagger、导出PDF、markdown、文档
---

## 需求

之前开发项目的时候，没有使用swagger来进行接口文档输出，导致开发完成的时候整理文档时，面对整个项目的接口，感觉到量多且都是无聊的重复劳动，所以干脆直接添加swagger，然后从swagger中直接生成接口文档。当然，如果想要输出的文档内容全一些，需要swagger中添加的注解也全一些才行。

## 实现

想要使用swagger，并能够导出文档，需要在pom文件中添加以下依赖，导出文档使用的是GitHub上的一个swagger2markup项目：

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <pjroject.reporting.outputEncoding>UTF-8</pjroject.reporting.outputEncoding>
  <java.version>1.8</java.version>
  <swagger2.version>2.7.0</swagger2.version>

  <swagger2markup.version>1.3.1</swagger2markup.version>
  <asciidoctor.input.directory>${project.basedir}/src/docs/asciidoc</asciidoctor.input.directory>
  <swagger.output.dir>${project.build.directory}/swagger</swagger.output.dir>
  <swagger.snippetOutput.dir>${project.build.directory}/asciidoc/snippets</swagger.snippetOutput.dir>
  <generated.asciidoc.directory>${project.build.directory}/asciidoc/generated</generated.asciidoc.directory>
  <asciidoctor.html.output.directory>${project.build.directory}/asciidoc/html</asciidoctor.html.output.directory>
  <asciidoctor.pdf.output.directory>${project.build.directory}/asciidoc/pdf</asciidoctor.pdf.output.directory>
  <swagger.input>${swagger.output.dir}/swagger.json</swagger.input>
</properties>

 <!-- swagger2  -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>${swagger2.version}</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>${swagger2.version}</version>
</dependency>

<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-staticdocs</artifactId>
  <version>2.4.0</version>
</dependency>

<dependency>
  <groupId>io.github.swagger2markup</groupId>
  <artifactId>swagger2markup</artifactId>
  <version>${swagger2markup.version}</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.asciidoctor</groupId>
  <artifactId>asciidoctorj-pdf</artifactId>
  <version>1.5.0-alpha.10.1</version>
  <scope>test</scope>
</dependency>

<plugin>
  <groupId>io.github.swagger2markup</groupId>
  <artifactId>swagger2markup-maven-plugin</artifactId>
  <version>1.3.1</version>
  <configuration>
    <swaggerInput>http://localhost:9999/v2/api-docs</swaggerInput>
    <outputDir>src/docs/asciidoc/generated</outputDir>
    <config>
      <swagger2markup.markupLanguage>ASCIIDOC</swagger2markup.markupLanguage>
    </config>
  </configuration>
</plugin>
<plugin>
  <groupId>org.asciidoctor</groupId>
  <artifactId>asciidoctor-maven-plugin</artifactId>
  <version>1.5.6</version>
  <configuration>
    <sourceDirectory>src/docs/asciidoc/generated</sourceDirectory>
    <outputDirectory>src/docs/asciidoc/html</outputDirectory>
    <backend>html</backend>
    <sourceHighlighter>coderay</sourceHighlighter>
    <attributes>
      <toc>left</toc>
    </attributes>
  </configuration>
</plugin>

<repository>
  <snapshots>
    <enabled>true</enabled>
    <updatePolicy>always</updatePolicy>
  </snapshots>
  <id>jcenter-releases</id>
  <name>jcenter</name>
  <url>http://jcenter.bintray.com</url>
</repository>
```

然后书写swagger的配置项：

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {

        ParameterBuilder cookiePar = new ParameterBuilder();
        List<Parameter> pars = new ArrayList<>();
        cookiePar.name("Cookie").description("Cookie信息").modelRef(new ModelRef("string")).parameterType("header").required(false)
                .defaultValue("xxx");
        pars.add(cookiePar.build());

        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // 自行修改为自己的包路径
                .apis(RequestHandlerSelectors.basePackage("cn.hyperchain.pboc.controller"))
                .paths(PathSelectors.any())
                .build()
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("api文档")
                .description("description")
                .version("1.0")
                .build();
    }
}
```

swagger数据默认存储并读取于/v2/api-docs(如果按照上述配置设置的话)，如果想更改可以在application.yml配置文件中修改：

```yaml
# 更改 swagger-ui 数据的读取路径
springfox:
  documentation:
    swagger:
    	# 默认是 /v2/api-docs
      v2.path: /v2/api-docs
```

然后可以在Collector层写对应的swagger注解，来让swagger生成接口数据：

```java
@RestController
@Api(value = "description")				 //swagger用于描述类的注解
@RequestMapping("/xxxx")
public class BlockChainController {
  
  @ApiOperation(value = "description",notes = "description")	//swagger用于描述方法的注解
  @PostMapping("/save")
  //swagger用于描述返回值信息的注解
  @ApiResponses(value = {
    @ApiResponse(code = -9999,message = "系统错误",response = Result.class),
    @ApiResponse(code = -8888,message = "调用合约失败",response = Result.class),
    @ApiResponse(code = -99,message = "操作失败",response = Result.class),
    @ApiResponse(code = -7001,message = "Redis异常",response = Result.class)
  })
  public Result saveData(
    @ApiParam(value = "上链参数",required = true) @RequestBody ChainParam chainParam
      ) throws Exception {
          Result result = blockChainService.saveData(chainParam.getKey(), chainParam.getSignature(), chainParam.getBatchId());
          return result;
      }
}
```

对于参数是RequestBody的参数，可以在实体类中加对应的注解，这样swagger就可以更加友好的给出提示：

```java
public class ChainParam {

    @ApiModelProperty(notes = "redis取值key标识",example = "user:34")
    private String key;

    @ApiModelProperty(notes = "签名",example = "0104382928dc5879597658f0eeb21b53f561c4dcd771b2de8cc4a721c67cbe8fe46603b17b7dc4eaaf56255bd61920cb221d1a99aa8d294d9807cf4138efee1ff73d3046022100d77913e0ee030cfc15484c6fe3b1dfe87d3ddd48b90928a0bf72b351d228e3690221009856cfdfbf00213a88149f7674d018ef8f7b40004396d3041976e31b39715730")
    private String signature;

    @ApiModelProperty(notes = "用户老状态值(用户状态更改)",example = "1")
    private String oldStatus;

    @ApiModelProperty(notes = "用户address",example = "149E093A5050B56869676A575CBF9FC5489867E5")
    private String address;

    @ApiModelProperty(notes = "redis取值标识",example = "addHandover")
    private String flag;

    @ApiModelProperty(notes = "标识id",example = "26")
    private String id;

    @ApiModelProperty(notes = "操作类型",example = "0")
    private String type;

    @ApiModelProperty(notes = "上链批次id",example = "210")
    private String batchId;
}
```



然后书写将swagger页面展示的数据导出为文档（PDF、markdown）的代码，这里我直接在测试类中使用：

```java
import io.github.swagger2markup.GroupBy;
import io.github.swagger2markup.Language;
import io.github.swagger2markup.Swagger2MarkupConfig;
import io.github.swagger2markup.Swagger2MarkupConverter;
import io.github.swagger2markup.builder.Swagger2MarkupConfigBuilder;
import io.github.swagger2markup.markup.builder.MarkupLanguage;
import org.asciidoctor.cli.AsciidoctorInvoker;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.net.URL;
import java.nio.file.Paths;

/**
 * @author ZhXiQi
 * @Title:
 * @date 2019-04-26 14:20
 */
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SwaggerToDoc {
		//注意swagger-ui界面读取的数据是否有加/v1之类的前缀，默认swagger.json存放位置为/v2/api-docs
    private final static String url = "http://localhost:9999/v1/v2/api-docs";
    private final static String toFile = "src/docs/";
    /**
     * 生成AsciiDocs格式文档
     * @throws Exception
     */
    @Test
    public void generateAsciiDocs() throws Exception {
        //    输出Ascii格式
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.ASCIIDOC)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL(url))
                .withConfig(config)
                .build()
                .toFile(Paths.get(toFile+"asciidoc/generated/api"));
    }

    @Test
    public void generatePDF() {
        //样式
        String style = "pdf-style=E:\\themes\\theme.yml";
        //字体
        String fontsdir = "pdf-fontsdir=E:\\fonts";
        //需要指定adoc文件位置
        String adocPath = "E:\\all.adoc";
        AsciidoctorInvoker.main(new String[]{"-a",style,"-a",fontsdir,"-b","pdf",adocPath});
    }
    /**
     * 生成Markdown格式文档
     * @throws Exception
     */
    @Test
    public void generateMarkdownDocs() throws Exception {
        //    输出Markdown格式
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.MARKDOWN)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL(url))
                .withConfig(config)
                .build()
                .toFolder(Paths.get(toFile+"markdown/generated"));
    }

    /**
     * 生成Confluence格式文档
     * @throws Exception
     */
    @Test
    public void generateConfluenceDocs() throws Exception {
        //    输出Confluence使用的格式
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.CONFLUENCE_MARKUP)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL(url))
                .withConfig(config)
                .build()
                .toFolder(Paths.get(toFile+"confluence/generated"));
    }

    /**
     * 生成AsciiDocs格式文档,并汇总成一个文件
     * @throws Exception
     */
    @Test
    public void generateAsciiDocsToFile() throws Exception {
        //    输出Ascii到单文件
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.ASCIIDOC)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL(url))
                .withConfig(config)
                .build()
                .toFile(Paths.get(toFile+"asciidoc/generated/all"));
    }

    /**
     * 生成Markdown格式文档,并汇总成一个文件
     * @throws Exception
     */
    @Test
    public void generateMarkdownDocsToFile() throws Exception {
        //    输出Markdown到单文件
        Swagger2MarkupConfig config = new Swagger2MarkupConfigBuilder()
                .withMarkupLanguage(MarkupLanguage.MARKDOWN)
                .withOutputLanguage(Language.ZH)
                .withPathsGroupedBy(GroupBy.TAGS)
                .withGeneratedExamples()
                .withoutInlineSchema()
                .build();

        Swagger2MarkupConverter.from(new URL(url))
                .withConfig(config)
                .build()
                .toFile(Paths.get(toFile+"markdown/generated/all"));
    }
}

```

注意，读取数据源一定要正确，否则无法会提示无法读取数据源，如果不清楚，可以访问swagger页面（http://localhost:port/swagger-ui.html），多刷新几次，刷新重新读取数据的时候会有提示，如下图：

![swagger](/images/posts/swagger/swagger-ui.png)