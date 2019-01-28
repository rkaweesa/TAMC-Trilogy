# TAMC-Trilogy
Tests added for multiple classes (#1028)
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.notNullValue;
import static org.mockito.Matchers.anyMapOf;
import static org.mockito.Matchers.anyString;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

import com.firstrain.workbench.constants.AdminDefaultConstant;
import com.firstrain.workbench.logging.SourceDBLogger;
import com.firstrain.workbench.object.AutoSuggest;
import com.firstrain.workbench.object.CompanyInformation;
import java.io.BufferedReader;
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Iterator;
@@ -22,22 +24,36 @@
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.ss.usermodel.WorkbookFactory;
import org.junit.After;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ErrorCollector;
import org.junit.rules.ExpectedException;
import org.junit.rules.RuleChain;
import org.junit.runner.RunWith;
import org.mockito.Mock;
import org.mockito.internal.util.reflection.Whitebox;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;
import org.springframework.web.multipart.MultipartFile;
import com.firstrain.utils.JSONUtility;
import com.firstrain.web.jsonres.JSONResponse.ResStatus;
import com.firstrain.workbench.constants.AdminDefaultConstant;
import com.firstrain.workbench.constants.Constant;
import com.firstrain.workbench.logging.SourceDBLogger;
import com.firstrain.workbench.object.AutoSuggest;
import com.firstrain.workbench.object.CompanyIdentyRes;
import com.firstrain.workbench.object.CompanyInformation;
import com.firstrain.workbench.util.HttpClientUtil;

@RunWith(PowerMockRunner.class)
@PrepareForTest({
        SourceDBLogger.class,
        CompIdentificationManagerImpl.class})
        CompIdentificationManagerImpl.class,
        Constant.class
})
public class CompIdentificationManagerImplTest {

    private static final File RESOURCES_DIRECTORY = new File("test/resources/");
@@ -53,9 +69,14 @@
    private SourceDBLogger logger;
    @Mock
    private ServletContext servletContext;
    @Mock
    private MultipartFile multipartFile;

    private final ErrorCollector collector = new ErrorCollector();
    private final ExpectedException expectedException = ExpectedException.none();

    @Rule
    public final ErrorCollector collector = new ErrorCollector();
    public final RuleChain chain = RuleChain.outerRule(collector).around(expectedException);

    @BeforeClass
    public static void beforeClass() {
@@ -69,21 +90,11 @@ public static void beforeClass() {

    @Before
    public void setUp() {
        PowerMockito.mockStatic(SourceDBLogger.class);
        when(SourceDBLogger.getInstance("SERVICE",
                CompIdentificationManagerImpl.class.getName())).thenReturn(logger);
        PowerMockito.mockStatic(SourceDBLogger.class, Constant.class);
        when(SourceDBLogger.getInstance("SERVICE", CompIdentificationManagerImpl.class.getName())).thenReturn(logger);
        manager = new CompIdentificationManagerImpl();
    }

    @After
    public void afterClass() {
        File[] files = DIR.listFiles();
        for (File f : files) {
            f.delete();
        }
        DIR.delete();
    }

    @Test
    public void givenCreateWorkbookWhenCreateThenReturnsString() throws Exception {
        // Arrange
@@ -113,21 +124,34 @@ public void givenCreateWorkbookWhenCreateThenReturnsString() throws Exception {
                builder.append(cellIterator.next());
            }
        }
        final String expected = new StringBuilder()
                .append("Company Information ")
                .append("SEARCH ENTITY")
                .append("TYPE")
                .append("ALGORITHM")
                .append("RANKING")
                .append("COMPANY ID")
                .append("COMPANY NAME")
                .append("SEARCHTOKEN")
                .append("est")
                .append("COMPANY")
                .append("AutoSuggest")
                .append("1.0")
                .toString();
        final String expected = "Company Information " + "SEARCH ENTITY" + "TYPE" + "ALGORITHM" + "RANKING" + 
                "COMPANY ID" + "COMPANY NAME" + "SEARCHTOKEN" + "est" + "COMPANY" + "AutoSuggest" + "1.0";
        collector.checkThat(builder.toString(), equalTo(expected));
        collector.checkThat(actual, notNullValue());
    }

    @Test
    public void givenWhenParseFileAndGetSuggestedCompaniesThen() throws Exception {
        // Arrange
        expectedException.expect(Exception.class);
        expectedException.expectMessage("Response incorrect");
        // Arrange
        when(multipartFile.getOriginalFilename()).thenReturn("abc.txt");
        InputStream stream = new ByteArrayInputStream("stream string".getBytes(StandardCharsets.UTF_8));
        when(multipartFile.getInputStream()).thenReturn(stream);

        BufferedReader bufferedReader = mock(BufferedReader.class);
        when(bufferedReader.readLine()).thenReturn("line").thenReturn(null);
        PowerMockito.whenNew(BufferedReader.class).withAnyArguments().thenReturn(bufferedReader);
        HttpClientUtil httpClientUtil = mock(HttpClientUtil.class);
        Whitebox.setInternalState(manager, "httpClientUtil", httpClientUtil);

        CompanyIdentyRes companyIdentyRes = new CompanyIdentyRes();
        companyIdentyRes.setMessage("successfully");
        companyIdentyRes.setStatus(ResStatus.SUCCESS);
        String result = JSONUtility.serialize(companyIdentyRes);
        when(httpClientUtil.getPostResponse(anyMapOf(String.class, String.class), anyString())).thenReturn(result);
        // Act
        manager.parseFileAndGetSuggestedCompanies(multipartFile, session);
    }
}
