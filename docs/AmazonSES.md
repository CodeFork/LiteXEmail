# LiteXEmail
Abstract interface to implement any kind of basic email message services (e.g. SMTP, SendGrid, MailKit, Mailgun, MailChimp, AmazonSES, SendinBlue)


## Add a dependency

### Nuget

Run the nuget command for installing the client as,
* `Install-Package LiteX.Email.Core`
* `Install-Package LiteX.Email.AmazonSES`


## Configuration

**AppSettings**
```js
{
  //LiteX AmazonSES settings
  "AmazonSESConfig": {
    "AmazonSESAccessKey": "--- REPLACE WITH YOUR AmazonSESAccessKey ---",
    "AmazonSESSecretKey": "--- REPLACE WITH YOUR AmazonSESSecretKey ---"
  }
}
```

**Startup Configuration**
```cs
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // 1. Use default configuration from appsettings.json's 'AmazonSESConfig'
        services.AddLiteXAmazonSESEmail();

        //OR
        // 2. Load configuration settings using options.
        services.AddLiteXAmazonSESEmail(option =>
        {
            //option. = "";
        });

        //OR
        // 3. Load configuration settings on your own.
        // (e.g. appsettings, database, hardcoded)
        var amazonSESConfig = new AmazonSESConfig();
        services.AddLiteXAmazonSESEmail(amazonSESConfig);
    }
}
```


## Usage

**Controller or Business layer**
```cs
/// <summary>
/// Customer controller
/// </summary>
public class CustomerController : Controller
{
    #region Fields
    
    private readonly IEmailSender _emailSender;

    #endregion

    #region Ctor

    /// <summary>
    /// Ctor
    /// </summary>
    /// <param name="emailSender"></param>
    public CustomerController(IEmailSender emailSender)
    {
        _emailSender = emailSender;
    }

    #endregion

    #region Methods

    /// <summary>
    /// Send email to customer
    /// </summary>
    /// <param name="customer"></param>
    /// <returns></returns>
    public IActionResult SendEmailToCustomer(Customer customer)
    {
        try
        {
            string subject = "Welcome!",
            body = "Welcome to our website!",
            fromAddress = "from@example.com",
            fromName = "Yousite",
            toAddress = customer.Email,
            toName = customer.FirstName,
            replyToAddress = "reply@example.com",
            replyToName = "Yousite";

            IEnumerable<string> bcc = new List<string>() { "bcc@example.com" };
            IEnumerable<string> cc = new List<string>() { "cc@example.com" };
            //IEnumerable<Attachment> attachments = new List<Attachment>();

            var isSent = _emailSender.SendEmail(subject, body, fromAddress, fromName, toAddress, toName, replyToAddress, replyToName, bcc, cc);
        }
        catch (Exception ex)
        {

            return BadRequest(ex);
        }
        return Ok();
    }

    #endregion

    #region Utilities

    private IList<Customer> GetCustomers()
    {
        IList<Customer> customers = new List<Customer>();

        customers.Add(new Customer() { Id = 1, Username = "ashish", Email = "toaashishpatel@outlook.com" });

        return customers;
    }

    private Customer GetCustomerById(int id)
    {
        Customer customer = null;

        customer = GetCustomers().ToList().FirstOrDefault(x => x.Id == id);

        return customer;
    }

    public static byte[] StreamToByteArray(Stream input)
    {
        byte[] buffer = new byte[16 * 1024];
        using (MemoryStream ms = new MemoryStream())
        {
            int read;
            while ((read = input.Read(buffer, 0, buffer.Length)) > 0)
            {
                ms.Write(buffer, 0, read);
            }
            return ms.ToArray();
        }
    }

    #endregion
}
```


### Test

Attachment feature is not tested