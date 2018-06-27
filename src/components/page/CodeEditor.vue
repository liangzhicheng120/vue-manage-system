<template>
    <div id="container">
        <el-form ref="factory" :model="factory" :rules="rules" label-width="80px">
            <el-form-item label="类名" prop="name">
                <el-input v-model="factory.name" placeholder="Test"></el-input>
            </el-form-item>
            <el-form-item label="包名" prop="package">
                <el-input v-model="factory.package" placeholder="com.xxx.demo"></el-input>
            </el-form-item>
            <el-form-item label="表名" prop="table">
                <el-input v-model="factory.table" placeholder="T_NEW_BOOK"></el-input>
            </el-form-item>
            <el-form-item label="字段" prop="columns">
                <el-input type="textarea" v-model="factory.columns" :autosize="{ minRows: 7, maxRows: 7}"
                          :placeholder="column"></el-input>
            </el-form-item>
            <el-form-item label="类型" prop="type">
                <el-radio-group v-model="factory.type" @change="submit('factory')">
                    <el-radio label="Xml"></el-radio>
                    <el-radio label="Bean"></el-radio>
                    <el-radio label="Params"></el-radio>
                    <el-radio label="VO"></el-radio>
                    <el-radio label="Dao"></el-radio>
                    <el-radio label="DaoImpl"></el-radio>
                    <el-radio label="Service"></el-radio>
                    <el-radio label="ServiceImpl"></el-radio>
                    <el-radio label="DaoAndServiceXml"></el-radio>
                    <el-radio label="Controller"></el-radio>
                    <el-radio label="Test"></el-radio>
                    <el-radio label="DaoMysqlImplTest"></el-radio>
                    <el-radio label="ServiceImplTest"></el-radio>
                    <!--               <el-radio label="Reset"></el-radio>-->
                </el-radio-group>
            </el-form-item>
            <el-form-item label="新建">
                <el-input v-model="factory.alias"></el-input>
            </el-form-item>
            <el-form-item label="输出">
                <el-input type="textarea" v-model="factory.output" :autosize="{ minRows: 20, maxRows: 20}"></el-input>
            </el-form-item>
        </el-form>
    </div>
</template>

<script>

    export default {
        data() {
            return {
                column: 'FID\tint\n' +
                'FCATEID\tint\n' +
                'FNAME\tvarchar\n',
                factory: {
                    type: '',
                    name: '',
                    package: '',
                    table: '',
                    columns: '',
                    output: '',
                    alias: '',
                },
                property: {
                    'xml':
                        `<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="{0}">

        <resultMap type="{1}" id="{2}">\n{3}
        </resultMap>
        {4}
        {5}
        {6}
        {7}
        {8}
        {9}

        <sql id="columns">{10}
        </sql>

</mapper>`,
                    'int': 'int',
                    'varchar': 'String',
                    'tinyint': 'int',
                    'datetime': 'Date',
                    'char': 'String',
                    'long': 'long',
                    'text': 'String',
                },
                sql: {
                    'get': `
        <select id="get" resultMap="{0}">
            SELECT
            <include refid="columns" />
            FROM
            {1}
            WHERE FID = #{id}
        </select>`,
                    'getList': `
        <select id="getList" resultMap="{0}">
            SELECT
            <include refid="columns" />
            FROM
            {1}
            WHERE 1 = 1
            <if test="keyword != null">
                AND FNAME LIKE CONCAT('%', #{keyword}, '%')
            </if>
            ORDER BY
            FORDER DESC , FID DESC
            LIMIT #{start}, #{size}
        </select>`,
                    'count': `
        <select id="count{0}" resultType="int">
            SELECT COUNT(*)
            FROM
            {1}
            WHERE 1 = 1
            <if test="keyword != null">
                AND FNAME LIKE CONCAT('%', #{keyword}, '%')
            </if>
        </select>`,
                    'delete': `
        <delete id="delete">
            DELETE
            FROM
            {0}
            WHERE
            FID = #{id}
        </delete>`,
                    'add': `
        <insert id="add">
            INSERT INTO
            {0}
            ({1}
            )
            VALUES
            ({2}
            )
        </insert>`,
                    'update': `
        <update id="update">
            UPDATE
            {0}
            SET{1}
            WHERE
                FID = #{{2}.id}
        </update>`,
                },
                rules: {
                    type: [
                        {required: true, message: '请选择类型', trigger: 'change'}
                    ],
                    name: [
                        {required: true, message: '请输入名称', trigger: 'blur'},
                    ],
                    package: [
                        {required: true, message: '请输入包名', trigger: 'blur'},
                    ],
                    columns: [
                        {required: true, message: '请输入字段', trigger: 'blur'},
                    ],
                    table: [
                        {required: true, message: '请输入表名', trigger: 'blur'},
                    ],
                },
            }
        },
        mounted: function () {

        },
        methods: {
            submit(formName) {
                if (this.factory.type == 'Reset') {
                    this.reset();
                    return;
                }
                this.$refs[formName].validate((valid) => {  // 校验
                    if (valid) {
                        this.changeColumns(this.factory.columns);
                        this.run(this.factory.type);
                        this.createFileName(this.factory.type);
                    } else {
                        return false;
                    }
                });
            },
            run(n) {   // 执行
                switch (n) {
                    case 'Xml':
                        var namespace = this.capitalize(this.factory.name, 'U');
                        var type = this.factory.package.trim() + "." + namespace;
                        var id = this.capitalize(namespace, 'L').trim();
                        var table = this.factory.table.trim();
                        var Fcolumns = [];
                        var columns = [];
                        var updates = [];
                        var columnsList = this.strip(this.factory.columns).split('\n').map(function (item) {
                            item = item.split('\t');
                            Fcolumns.push(item[0]);
                            var column = this.camelCase(item[0].substring(1).toLowerCase());
                            columns.push('#{' + id + '.' + column + "}");
                            var property = this.property[item[1]];
                            var result = '\t\t<result column = "{0}" property = "{1}" />\n'.format(item[0], column);
                            updates.push(item[0] + '=#{' + id + '.' + column + '}');
                            return result;
                        }, this);
                        var result = this.strip(columnsList.join(''));
                        var get = this.sql.get.format(id, table);
                        var getList = this.sql.getList.format(id, table);
                        var count = this.sql.count.format(namespace, table);
                        var del = this.sql.delete.format(table);
                        var add = this.sql.add.format(table, Fcolumns.join(',\n\t\t'), columns.join(',\n\t\t'));
                        add = add.replace('FID,', '').replace("#{" + id + ".id" + "},", '');
                        add = add.replace("#{" + id + ".posttime" + "},", 'NOW(),');
                        add = add.replace("#{" + id + ".lmodify" + "}", 'NOW()');
                        var update = this.sql.update.format(table, updates.join(',\n\t\t'), id);
                        update = update.replace("FID=#{" + id + ".id},", "");
                        update = update.replace("#{" + id + ".posttime" + "},", 'NOW(),');
                        update = update.replace("#{" + id + ".lmodify" + "}", 'NOW()');
                        var output = this.property['xml'].format(namespace, type, id, result, get, getList, count, del, add, update, Fcolumns.join(','));
                        this.factory.output = output;
                        break;
                    case 'Bean':
                        this.factory.output = this.createBean(this.factory.name);
                        break;
                    case 'Params':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var impackage = 'import java.text.ParseException;\n' +
                            '\n' +
                            'import org.apache.commons.lang3.time.DateUtils;\n\n';
                        var head = '\tpublic {0} transformModel() throws ParseException {\n\t\t{1} {2} = new {3}();\n\t'.format(name, name, lname, name);
                        var body = [];
                        this.strip(this.factory.columns).split('\n').map(function (item) {
                            item = item.split('\t');
                            var property = this.camelCase(item[0].substring(1).toLowerCase());
                            var Uproperty = this.capitalize(property, 'U');
                            if (this.isContains(property, 'Time') || property == 'posttime' || property == 'lmodify') {
                                body.push('\t{0}.set{1}(DateUtils.parseDate(this.get{2}(), new String[] { "yyyy-MM-dd HH:mm:ss" }));'.format(lname, Uproperty, Uproperty));
                            } else {
                                body.push('\t{0}.set{1}(this.get{2}());'.format(lname, Uproperty, Uproperty));
                            }
                        }, this);
                        var foot = '\n\t\treturn {0};\n\t}\n}'.format(lname);
                        var transformModel = head + body.join('\n\t') + foot;
                        var bean = this.createBean(this.factory.name + 'AdminParam');
                        var output = impackage + bean.substring(0, bean.length - 1) + transformModel;
                        this.factory.output = output;
                        break;
                    case 'VO':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var vo = name + 'AdminVO';
                        var lvo = this.capitalize(vo, 'L');
                        var impackage = 'import org.apache.commons.lang3.time.DateFormatUtils;\n\n';
                        var head = '\tpublic static {0} build({1} {2}) {\n\t\t{3} {4} = new {5}();\n\t'.format(vo, name, lname, vo, lvo, vo);
                        var body = [];
                        this.strip(this.factory.columns).split('\n').map(function (item) {
                            item = item.split('\t');
                            var property = this.camelCase(item[0].substring(1).toLowerCase());
                            var Uproperty = this.capitalize(property, 'U');
                            if (this.isContains(property, 'Time') || property == 'posttime' || property == 'lmodify') {
                                body.push('\t{0}.set{1}(DateFormatUtils.format({2}.get{3}(), "yyyy-MM-dd HH:mm:ss"));'.format(lvo, Uproperty, lname, Uproperty));
                            } else {
                                body.push('\t{0}.set{1}({2}.get{3}());'.format(lvo, Uproperty, lname, Uproperty));
                            }
                        }, this);
                        var foot = '\n\t\treturn {0};\n\t}\n}'.format(lvo);
                        var build = head + body.join('\n\t') + foot;
                        var bean = this.createBean(this.factory.name + 'AdminVO');
                        var output = impackage + bean.substring(0, bean.length - 1) + build;
                        this.factory.output = output;
                        break;
                    case 'Dao':
                        this.daoOrService('Dao');
                        break;
                    case 'DaoImpl':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var impackage = 'import java.util.HashMap;\n' +
                            'import java.util.List;\n' +
                            'import java.util.Map;\n' +
                            'import com.meizu.cal.utils.SqlUtils;\n\n';
                        var head = 'public class {0}DaoMysqlImpl extends BaseDao implements {1}Dao {\n'.format(name, name);
                        var add =
                            `\n    @Override
    public void add({0} {1}) {
        Map<String, Object> param = new HashMap<String, Object>();
        param.put("{2}", {3});
        this.getSqlSession().insert("{4}.add", param);
    }\n`.format(name, lname, lname, lname, name);
                        var del =
                            `\n    @Override
    public void delete(Long id) {
        Map<String, Object> param = new HashMap<String, Object>();
        param.put("id", id);
        this.getSqlSession().delete("{0}.delete", param);
    }\n`.format(name);
                        var getList =
                            `\n    @Override
    public List<{0}> getList(String keyword, int page, int size) {
        Map<String, Object> param = new HashMap<String, Object>();
        param.put("keyword", SqlUtils.escapeSQLLike(keyword));
        param.put("start", (page - 1) * size);
        param.put("size", size);
        return this.getSqlSession().selectList("{1}.getList", param);
    }\n`.format(name, name);
                        var count =
                            `\n    @Override
    public int count{0}(String keyword) {
        Map<String, Object> param = new HashMap<String, Object>();
        param.put("keyword", SqlUtils.escapeSQLLike(keyword));
        return this.getSqlSession().selectOne("{1}.count{2}", param);
    }\n`.format(name, name, name);
                        var update =
                            `\n    @Override
    public void update({0} {1}) {
        Map<String, Object> param = new HashMap<String, Object>();
        param.put("{2}", {3});
        this.getSqlSession().update("{4}.update", param);
    }\n`.format(name, lname, lname, lname, name);
                        var get =
                            `\n    @Override
    public {0} get(Long id) {
        Map<String, Object> param = new HashMap<String, Object>();
        param.put("id", id);
        return this.getSqlSession().selectOne("{1}.get", param);
    }\n`.format(name, name);
                        var foot = '}';
                        var output = impackage + head + add + del + getList + count + update + get + foot;
                        this.factory.output = output;
                        break;
                    case 'Service':
                        this.daoOrService('Service');
                        break;
                    case 'ServiceImpl':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var impackage = 'import java.util.List;\n' +
                            '\n' +
                            'import org.springframework.beans.factory.annotation.Autowired;\n\n';
                        var head = 'public class {0}ServiceImpl implements {1}Service {\n'.format(name, name);
                        var dao =
                            `\n    @Autowired
    private {0}Dao {1}Dao;\n`.format(name, lname);
                        var add =
                            `\n    @Override
    public void add({0} {1}) {
        {2}Dao.add({3});
    }\n`.format(name, lname, lname, lname);
                        var del =
                            `\n    @Override
    public void delete(Long id) {
        {0}Dao.delete(id);
    }\n`.format(lname);
                        var getList =
                            `\n    @Override
    public List<{0}> getList(String keyword, int page, int size) {
        return {1}Dao.getList(keyword, page, size);
    }\n`.format(name, lname);
                        var count =
                            `\n    @Override
    public int count{0}(String keyword) {
        return {1}Dao.count{2}(keyword);
    }\n`.format(name, lname, name);
                        var update =
                            `\n    @Override
    public void update({0} {1}) {
        {2}Dao.update({3});
    }\n`.format(name, lname, lname, lname);
                        var get =
                            `\n    @Override
    public {0} get(Long id) {
        return {1}Dao.get(id);
    }\n`.format(name, lname);
                        var foot = '}';
                        var output = impackage + head + dao + add + del + getList + count + update + get + foot;
                        this.factory.output = output;
                        break;
                    case 'Controller':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var importPackage = 'import java.text.ParseException;\n' +
                            'import java.util.ArrayList;\n' +
                            'import java.util.List;\n' +
                            '\n' +
                            'import org.springframework.beans.factory.annotation.Autowired;\n' +
                            'import org.springframework.stereotype.Controller;\n' +
                            'import org.springframework.web.bind.annotation.RequestMapping;\n' +
                            'import org.springframework.web.bind.annotation.RequestParam;\n' +
                            'import org.springframework.web.bind.annotation.ResponseBody;\n' +
                            '\n' +
                            'import com.alibaba.fastjson.JSONObject;\n' +
                            'import com.meizu.cal.model.constants.BaseResultModel;\n' +
                            'import com.meizu.cal.utils.CheckUtil;\n' +
                            'import com.meizu.cal.web.log.OperationLog;\n\n';
                        var head =
                            `@Controller
@RequestMapping(value = "/admin/{0}")
public class {1}Controller {\n`.format(lname.toLowerCase(), name);
                        var service =
                            `\n    @Autowired
    private {0}Service {1}Service;\n`.format(name, lname);
                        var list =
                            `\n    @RequestMapping("/list")
    @ResponseBody
    public BaseResultModel get{0}List(@RequestParam(required = false) String keyword,int page, int size) {
        List<{1}> {2}s = {3}Service.getList(keyword, page, size);
        List<{4}AdminVO> {5}AdminVOs = new ArrayList<{6}AdminVO>();
        for ({7} {8} : {9}s) {
            {10}AdminVOs.add({11}AdminVO.build({12}));
        }
        JSONObject value = new JSONObject();
        value.put("data", {13}AdminVOs);
        value.put("total", {14}Service.count{15}(keyword));
        BaseResultModel baseResultModel = new BaseResultModel();
        baseResultModel.setValue(value);
        return baseResultModel;
    }\n`.format(name, name, lname, lname, name, lname, name, name, lname, lname, lname, name, lname, lname, lname, name);
                        var add =
                            `\n    @RequestMapping("/add")
    @ResponseBody
    @OperationLog
    public BaseResultModel add{0}({1}AdminParam {2}AdminParam) throws ParseException{
        {3}Service.add({4}AdminParam.transformModel());
        BaseResultModel baseResultModel = new BaseResultModel();
        baseResultModel.setValue(Boolean.TRUE);
        return baseResultModel;
    }\n`.format(name, name, lname, lname, lname);
                        var del =
                            `\n    @RequestMapping("/del")
    @ResponseBody
    @OperationLog
    public BaseResultModel delte{0}(Long id) {
        {1}Service.delete(id);
        BaseResultModel baseResultModel = new BaseResultModel();
        baseResultModel.setValue(Boolean.TRUE);
        return baseResultModel;
    }\n`.format(name, lname);
                        var update =
                            `\n    @RequestMapping(value = "/update")
    @ResponseBody
    @OperationLog
    public BaseResultModel update{0}({1}AdminParam {2}AdminParam) throws ParseException{
        CheckUtil.checkLLeZero({5}AdminParam.getId(), "id 不能为空");
        {3}Service.update({4}AdminParam.transformModel());
        BaseResultModel baseResultModel = new BaseResultModel();
        baseResultModel.setValue(Boolean.TRUE);
        return baseResultModel;
    }\n`.format(name, name, lname, lname, lname, lname);
                        var get =
                            `\n    @RequestMapping("/get")
    @ResponseBody
    public BaseResultModel get{0}(Long id) {
        CheckUtil.checkLLeZero({6}AdminParam.getId(), "id 不能为空");
        {1} {2} = {3}Service.get(id);
        BaseResultModel baseResultModel = new BaseResultModel();
        baseResultModel.setValue({4}AdminVO.build({5}));
        return baseResultModel;
    }\n`.format(name, name, lname, lname, name, lname, lname);
                        var foot = '}';
                        var output = importPackage + head + service + list + add + del + update + get + foot;
                        this.factory.output = output;
                        break;
                    case 'DaoMysqlImplTest':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        this.factory.alias = '{0}DaoMysqlImplTest'.format(name);
                        var table = this.factory.table.trim();
                        var impackage = 'import java.util.List;\n' +
                            '\n' +
                            'import org.springframework.beans.factory.annotation.Autowired;\n' +
                            'import org.testng.Assert;\n' +
                            'import org.testng.annotations.AfterClass;\n' +
                            'import org.testng.annotations.BeforeClass;\n' +
                            'import org.testng.annotations.Test;\n\n';
                        var body =
                            `public class {0}DaoMysqlImplTest extends TestDaoBase {

	@Autowired
	private {1}DaoMysqlImpl {2}DaoMysqlImpl;

	@BeforeClass
	public void beforeClass() {
		this.init("{3}");
	}

	@AfterClass
	public void afterClass() {
		this.deleteFromTables("{4}");
	}\n\n`.format(name, name, lname, table, table);

                        var testAdd = `	@Test
	public void testAdd() throws Exception {
		{0} {1} = CreateBean.newInstance({2}.class);
		{4}DaoMysqlImpl.add({5});
		Assert.assertEquals(this.countRowsInTable("{6}"), 2);
	}\n\n`.format(name, lname, name, lname, lname, table);

                        var testDelete = `	@Test
	public void testDelete() throws Exception {
		Long id = 1L;
		{0}DaoMysqlImpl.delete(id);
		Assert.assertEquals(this.countRowsInTable("{1}"), 0);
	}\n\n`.format(lname, table);

                        var testGetList = `	@Test
	public void testGetList() throws Exception {
		String keyword = null;
		Integer page = 1;
		Integer size = 1;
		List<{0}> result = {1}DaoMysqlImpl.getList(keyword, page, size);
		Assert.assertEquals(result.size(), 1);
	}\n\n`.format(name, lname);

                        var testCount = `	@Test
	public void testCount{0}() throws Exception {
		String keyword = null;
		int result = {1}DaoMysqlImpl.count{2}(keyword);
		Assert.assertEquals(result, 1);
	}\n\n`.format(name, lname, name);

                        var testUpdate = `	@Test
	public void testUpdate() throws Exception {
		{0} {1} = CreateBean.newInstance({2}.class);
		{3}DaoMysqlImpl.update({4});
		Assert.assertEquals(this.countRowsInTable("{5}"), 1);
	}\n\n`.format(name, lname, name, lname, lname, table);

                        var testGet = `	@Test
	public void testGet() throws Exception {
		Long id = 1L;
		{0} result = {1}DaoMysqlImpl.get(id);
		Assert.assertNotNull(result);
	}\n\n`.format(name, lname);

                        var foot = '}'

                        this.factory.output = impackage + body + testAdd + testCount + testDelete + testGetList + testUpdate + testGet + foot;
                        break;
                    case 'DaoAndServiceXml':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var packageName = this.factory.package.trim();
                        var dao = `    <bean id="{0}Dao" class="{1}.{2}DaoMysqlImpl">
	<property name="sqlSessionFactory" ref="calSqlSessionFactory"></property>
    </bean>\n\n\n`.format(lname, packageName, name);
                        var service = `    <bean id="{0}ServiceImpl" class="{1}.{2}ServiceImpl" />`.format(lname, packageName, name);
                        var output = dao + service;
                        this.factory.output = output;
                        break;
                    case 'ServiceImplTest':
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var importPackage = 'import java.util.List;\n' +
                            'import java.util.ArrayList;\n' +
                            '\n' +
                            'import org.testng.Assert;\n' +
                            'import org.testng.annotations.Test;\n' +
                            '\n' +
                            'import mockit.Expectations;\n' +
                            'import mockit.Injectable;\n' +
                            'import mockit.Tested;\n' +
                            'import mockit.VerificationsInOrder;\n\n';
                        var head = 'public class {0}ServiceImplTest {\n\n'.format(name);
                        var dao = '    @Injectable\n' +
                            '    private {0}Dao {1}Dao;\n\n'.format(name, lname);
                        var service = '    @Tested\n' +
                            '    private {0}ServiceImpl {1}ServiceImpl;\n\n'.format(name, lname);
                        var add = `    @Test
    public void testAdd() throws Exception {
        final {0} {1} = new {2}();
        {3}ServiceImpl.add({4});
        new VerificationsInOrder() {
            {
                {5}Dao.add({6});
            }
        };
    }\n\n`.format(name, lname, name, lname, lname, lname, lname);
                        var del = `    @Test
    public void testDelete() throws Exception {
        Long id = 1L;
        {0}ServiceImpl.delete(id);
        new VerificationsInOrder() {
            {
                {1}ServiceImpl.delete(anyLong);
            }
        };
    }\n\n`.format(lname, lname);
                        var getList = `    @Test
    public void testGetList() throws Exception {
        final List<{0}> {1}s = new ArrayList<{2}>();
        new Expectations() {
            {
                {3}Dao.getList(anyString, anyInt, anyInt);
                result = {4}s;
            }
        };
        String keyword = null;
        int page = 1;
        int size = 1;
        List<{5}> result = {6}ServiceImpl.getList(keyword, page, size);
        Assert.assertSame(result, {7}s);
    }\n\n`.format(name, lname, name, lname, lname, name, lname, lname);
                        var count = `    @Test
    public void testCount{0}() throws Exception {
        final int count = 0;
        new Expectations() {
            {
                {1}Dao.count{2}(anyString);
            }
        };
        String keyword = null;
        int result = {3}ServiceImpl.count{4}(keyword);
        Assert.assertSame(result, count);
    }\n\n`.format(name, lname, name, lname, name);
                        var update = `    @Test
    public void testUpdate() throws Exception {
        final {0} {1} = new {2}();
        {3}ServiceImpl.update({4});
        new VerificationsInOrder() {
            {
                {5}ServiceImpl.update({6});
            }
        };
    }\n\n`.format(name, lname, name, lname, lname, lname, lname);
                        var get = `    @Test
    public void testGet() throws Exception {
        final {0} {1} = new {2}();
        new Expectations() {
            {
                {3}Dao.get(anyLong);
                result = {4};
            }
        };
        Long id = 1L;
        {5} result = {6}ServiceImpl.get(id);
        Assert.assertSame(result, {7});
    }\n\n`.format(name, lname, name, lname, lname, name, lname, lname);
                        var foot = '}'
                        var output = importPackage + head + dao + service + add + del + getList + count + update + get + foot;
                        this.factory.output = output;
                        break;
                    case 'Test':
                        this.factory.alias = '';
                        var get = this.getter(this.factory.name);
                        var colunm = this.getColunm();
                        var name = this.capitalize(this.factory.name, 'U');
                        var lname = this.capitalize(name, 'L');
                        var output =
                            `    map.put("{0}", new Runnable() {
            @Override
            public void run() {
                TestDaoBase.this.deleteFromTables("{1}");

                {2} {3} = CreateBean.newInstance({4}.class);
                {5}.setId(1L);
                TestDaoBase.this.jdbcTemplate
                        .update("INSERT INTO {6} ({7})"
                                + "VALUES ({8})",
                                new Object[] { {9} });
            }
        });`.format(this.factory.table, this.factory.table, name, lname, name, lname,
                                this.factory.table, colunm.join(','), this.addMark(colunm).join(','), get.join(',')
                            )
                        this.factory.output = output;
                        break;
                }
            },
            reset() {   // 重置
                this.factory.name = '';
                this.factory.columns = '';
                this.factory.output = '';
                this.factory.package = '';
                this.factory.table = '';
                this.factory.alias = '';
            },
            capitalize(str, type) {   // 首字符大写
                try {
                    var str = str.trim();
                    if (type == 'L') {
                        return str.replace(/\b(\w)|\s(\w)/g, function (m) {
                            return m.toLowerCase();
                        });
                    }
                    if (type == 'U') {
                        return str.replace(/\b(\w)|\s(\w)/g, function (m) {
                            return m.toUpperCase();
                        });
                    }
                }
                catch (e) {

                }
            },
            camelCase(str) {  // 驼峰命名法
                if (str.indexOf('_') > 0) {
                    str = str.split('_')[0] + this.capitalize(str.split('_')[1], 'U')
                        + this.capitalize(str.split('_')[2], 'U')
                        + this.capitalize(str.split('_')[3], 'U')
                        + this.capitalize(str.split('_')[4], 'U')
                        + this.capitalize(str.split('_')[5], 'U');
                }
                return str.replace(new RegExp('undefined', 'gm'), '');
            },
            strip(str) {
                return str.replace(/^\n+|\n+$/g, "");
            },
            createBean(name) {
                var name = this.capitalize(name, 'U').trim();
                var attribute = [];
                var getSet = [];
                this.strip(this.factory.columns).split('\n').map(function (item) {
                    item = item.split('\t');
                    var property = this.camelCase(item[0].substring(1).toLowerCase());
                    var type = this.property[item[1]];
                    if (this.isContains(name, 'VO') || this.isContains(name, 'Param')) {
                        if (this.isContains(property, 'Time') || property == 'posttime' || property == 'lmodify') {
                            type = 'String';
                        }
                    }
                    attribute.push('private {0} {1};\n'.format(type, property));
                    var get = 'public {0} get{1}()'.format(type, this.capitalize(property, 'U')) + '{\n\t\t' + 'return {0};\n\t'.format(property) + '}\n';
                    var set = 'public void set{0}({1} {2})'.format(this.capitalize(property, 'U'), type, property) + '{\n\t\t' + 'this.{0} = {1};\n\t'.format(property, property) + '}\n';
                    getSet.push(get);
                    getSet.push(set);
                }, this);
                var bean = 'public class {0}'.format(name) + '{\n\n\t' + attribute.join('\n\t') + '\n\t' + getSet.join('\n\t') + '\n}';
                return bean;
            },
            daoOrService(type) {
                var name = this.capitalize(this.factory.name, 'U');
                var lname = this.capitalize(name, 'L');
                var className = (type == 'Service') ? name + 'Service' : name + 'Dao';
                var head = 'public interface {0} {\n'.format(className);
                var add = '\n\tvoid add({0} {1});\n'.format(name, lname);
                var del = '\n\tvoid delete(Long id);\n';
                var getList = '\n\tList<{0}> getList(String keyword,int page,int size);\n'.format(name);
                var count = '\n\tint count{0}(String keyword);\n'.format(name);
                var update = '\n\tvoid update({0} {1});\n'.format(name, lname);
                var get = '\n\t{0} get(Long id);\n'.format(name);
                var foot = '}';
                var impackage = 'import java.util.List;\n\n';
                var serviceimpackage = '';
                var output = impackage + head + add + del + getList + count + update + get + foot;
                this.factory.output = output;
            },
            isContains(str1, str2) {
                return str1.indexOf(str2) >= 0;
            },
            getter(name) {
                var result = [];
                this.strip(this.factory.columns).split('\n').map(function (item) {
                    result.push('{0}.get{1}()'.format(this.capitalize(name, 'L'), this.capitalize(this.camelCase(item.split('\t')[0].substring(1).toLowerCase()), 'U')));
                }, this);
                return result;
            },
            getColunm() {
                var result = [];
                this.strip(this.factory.columns).split('\n').map(function (item) {
                    result.push(item.split('\t')[0]);
                }, this);
                return result;
            },
            addMark(arr) {
                var result = [];
                arr.map(function (item) {
                    result.push('?');
                })
                return result;
            },
            createFileName: function (type) {
                var name = this.capitalize(this.factory.name, 'U');
                switch (type) {
                    case 'Xml':
                        this.factory.alias = name;
                        break;
                    case 'Bean':
                        this.factory.alias = name;
                        break;
                    case 'Params':
                        this.factory.alias = '{0}AdminParam'.format(name);
                        break;
                    case 'VO':
                        this.factory.alias = '{0}AdminVO'.format(name);
                        break;
                    case 'Dao':
                        this.factory.alias = '{0}Dao'.format(name);
                        break;
                    case 'DaoImpl':
                        this.factory.alias = '{0}DaoMysqlImpl'.format(name);
                        break;
                    case 'Service':
                        this.factory.alias = '{0}Service'.format(name);
                        break;
                    case 'ServiceImpl':
                        this.factory.alias = '{0}ServiceImpl'.format(name);
                        break;
                    case 'Controller':
                        this.factory.alias = '{0}Controller'.format(name);
                        break;
                    case 'ServiceImplTest':
                        this.factory.alias = '{0}ServiceImplTest'.format(name);
                        break;
                }
            },
            changeColumns: function (column) {
                var _this = this;
                var columns = column.split('\n')
                var new_columns = new Array();
                columns.map(function (item) {
                    if (_this.isContains(item, 'ID')) {
                        new_columns.push(item.replace('int', 'long'))
                    } else {
                        new_columns.push(item)
                    }
                })
                this.factory.columns = new_columns.join('\n');
            },
        }
    }
</script>
