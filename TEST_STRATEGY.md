package example;

import com.microsoft.playwright.*;
import org.testng.Assert;
import org.testng.ITestResult;
import org.testng.annotations.*;

import java.io.File;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Paths;
import java.text.SimpleDateFormat;
import java.util.Date;

public class PlaystationTest {

    private static Playwright playwright;
    private static Browser browser;
    private BrowserContext context;
    private Page page;

    @BeforeClass
    public static void setUpPlaywright() {
        playwright = Playwright.create();
        // Por defecto corre en Headless (CI/CD), se puede cambiar con -Dheadless=false
        boolean headless = Boolean.parseBoolean(System.getProperty("headless", "true"));
        browser = playwright.chromium().launch(new BrowserType.LaunchOptions().setHeadless(headless));
    }

    @BeforeMethod
    public void createContext(ITestResult result) {
        context = browser.newContext();
        page = context.newPage();
        result.setAttribute("page", page);
        page.navigate("https://www.liverpool.com.mx/tienda/home");


    /**
     * En caso de Llamado al driver y ejecutable de Firefox
     */

    @BeforeMethod
    public void setup() {
        System.setProperty("webdriver.gecko.driver", "C:\\Users\\Drivers_QA\\FireFox\\geckodriver.exe");
        FirefoxOptions options = new FirefoxOptions();
        options.setBinary("C:\\Program Files\\Mozilla Firefox\\firefox.exe");
        driver = new FirefoxDriver(options);
        driver.manage().window().maximize();
        driver.get("https://www.liverpool.com.mx/tienda/home");
    }

        
    }

    /**
     * Método para leer el término de búsqueda desde un archivo .txt externo
     */
    private String obtenerTerminoBusqueda() {
        String rutaArchivo = "busqueda.txt"; // Ruta en la raíz del proyecto
        String terminoDefault = "Playstation 5"; // Valor por defecto en caso de error

        try (BufferedReader reader = new BufferedReader(new FileReader(rutaArchivo))) {
            String linea = reader.readLine();
            if (linea != null && !linea.trim().isEmpty()) {
                return linea.trim();
            }
        } catch (IOException e) {
            System.err.println("No se pudo leer 'busqueda.txt'. Usando valor por defecto: " + terminoDefault);
        }
        return terminoDefault;
    }

    /**
     * Método para tomar evidencias manuales durante la prueba
     * @param nombreTexto Identificador de la captura (ej: "HOME", "BUSQUEDA")
     */
    public void takeScreenshotDuringTest(String nombreTexto) {
        try {
            // 1. Obtener la fecha y hora actual (formato: yyyyMMdd_HHmmss)
            String fechaHora = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());

            // 2. Definir carpeta de destino
            String carpetaDestino = "C:\\Evidencias\\";
            File directorio = new File(carpetaDestino);
            if (!directorio.exists()) {
                directorio.mkdirs();
            }

            // 3. Formar la ruta completa del archivo
            String rutaArchivo = carpetaDestino + nombreTexto + "_" + fechaHora + ".png";

            // 4. Guardar la captura usando la API de Playwright
            page.screenshot(new Page.ScreenshotOptions()
                    .setPath(Paths.get(rutaArchivo))
                    .setFullPage(true));

            System.out.println("Evidencia guardada en: " + rutaArchivo);

        } catch (Exception e) {
            System.err.println("Error al guardar la captura: " + e.getMessage());
        }
    }

    @Test
    public void testBuscarYExtraerPlaystation() {
        // Obtenemos el término parametrizado desde el archivo .txt
        String terminoBusqueda = obtenerTerminoBusqueda();
        System.out.println("Iniciando búsqueda parametrizada para: " + terminoBusqueda);

        // Tomar evidencia en la página principal
        takeScreenshotDuringTest("HOME");

        // 1. Búsqueda
        page.locator("[data-testid='blt26617d4f2e17657d-header-search-input']").click();
        page.locator("[data-testid='blt26617d4f2e17657d-header-search-input']").fill(terminoBusqueda);
        page.locator("[data-testid='blt26617d4f2e17657d-header-search-input']").press("Enter");

        takeScreenshotDuringTest("RESULTADOS_BUSQUEDA");

        // 2. Filtro y ordenamiento
        page.locator("//button[@data-testid='tiles-navigations-button-sticky-button'][.//span[text()='BLANCO']]").click();
        page.locator("//button[@data-testid='plp-page-sort-button'][.//span[text()='Ordenar']]").click();
        page.locator("//span[normalize-space()='Menor precio']").click();

        takeScreenshotDuringTest("FILTROS_APLICADOS");

        // 3. Extracción de los primeros 5 productos
        Locator productos = page.locator("//li[contains(@class, 'm-product__card')]");
        int limite = Math.min(5, productos.count());

        System.out.println("--- PRIMEROS " + limite + " RESULTADOS PARA '" + terminoBusqueda + "' ---");
        for (int i = 0; i < limite; i++) {
            String nombre = productos.nth(i).locator("h5, h4, p.title").textContent();
            String precio = productos.nth(i).locator("p.a-card-discount, p.a-card-real").textContent();
            System.out.println("Producto #" + (i + 1) + ": " + nombre + " | Precio: " + precio);
        }

        Assert.assertTrue(limite > 0, "Se encontraron productos en la búsqueda");


        //En el txt se ingreasa el campo de busqueda a iterar
            //TXT TerminoDefault =Playstation 5

    }

    @AfterMethod
    public void tearDown(ITestResult result) {
        // Captura automática únicamente en caso de fallo (Gestión del Framework)
        if (result.getStatus() == ITestResult.FAILURE) {
            String screenshotPath = "target/screenshots/" + result.getName() + ".png";
            page.screenshot(new Page.ScreenshotOptions().setPath(Paths.get(screenshotPath)).setFullPage(true));
            System.out.println("Fallo detectado. Captura guardada en: " + screenshotPath);
        }
        context.close();
    }

    @AfterClass
    public static void tearDownAll() {
        browser.close();
        playwright.close();
    }
}
