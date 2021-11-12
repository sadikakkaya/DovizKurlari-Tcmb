namespace WindowsConsoleDoviz
{
    public class Program
    {
        private static string zaman = ConfigurationManager.AppSettings["CalismaSuresi"];
        private static string kurmailler = ConfigurationManager.AppSettings["KurMaileri"];
        private static string gondericikul = ConfigurationManager.AppSettings["GondericiMailKullaniciAdi"];
        private static string gondericisifre = ConfigurationManager.AppSettings["GondericiMailSifre"];
        private static string gondericibaslik = ConfigurationManager.AppSettings["GondericiBaslik"];
        private static string _MailServerPort = ConfigurationManager.AppSettings["MailServerPort"];
        private static string _MailServerSmtp = ConfigurationManager.AppSettings["MailServerSmtp"];

        static void Main(string[] args)
        {

            List<DovizKurlari> DovizList = new List<DovizKurlari>();
          
                XDocument xDoc = XDocument.Load("https://www.tcmb.gov.tr/kurlar/today.xml");
                DovizList = xDoc.Descendants("Currency").Where(x => (string)x.Attribute("Kod") == "USD" || (string)x.Attribute("Kod") == "EUR" || (string)x.Attribute("Kod") == "GBP" || (string)x.Attribute("Kod") == "SAR")
                        .Select(o => new DovizKurlari
                        {
                            Kod = (string)o.Attribute("Kod"),
                            Ad = (string)o.Element("Isim"),
                            AlisFiyat = (string)o.Element("BanknoteBuying"),
                            SatisFiyat = (string)o.Element("BanknoteSelling"),
                        })
                        .ToList();

            string _alis_USD = DovizList.Where(x => x.Kod == "USD").Select(x => x.AlisFiyat).FirstOrDefault().ToString();
            string _alis_EUR = DovizList.Where(x => x.Kod == "EUR").Select(x => x.AlisFiyat).FirstOrDefault().ToString();
            string _alis_GBP = DovizList.Where(x => x.Kod == "GBP").Select(x => x.AlisFiyat).FirstOrDefault().ToString();
            string _alis_SAR = DovizList.Where(x => x.Kod == "SAR").Select(x => x.AlisFiyat).FirstOrDefault().ToString();
            string _satis_USD = DovizList.Where(x => x.Kod == "USD").Select(x => x.SatisFiyat).FirstOrDefault().ToString();
            string _satis_EUR = DovizList.Where(x => x.Kod == "EUR").Select(x => x.SatisFiyat).FirstOrDefault().ToString();
            string _satis_GBP = DovizList.Where(x => x.Kod == "GBP").Select(x => x.SatisFiyat).FirstOrDefault().ToString();
            string _satis_SAR = DovizList.Where(x => x.Kod == "SAR").Select(x => x.SatisFiyat).FirstOrDefault().ToString();

            Doviz_MailGonder(_alis_USD, _alis_EUR, _alis_GBP, _alis_SAR, _satis_USD, _satis_EUR, _satis_GBP, _satis_SAR);

        }
        private static void Doviz_MailGonder(string _alis_USD, string _alis_EUR, string _alis_GBP, string _alis_SAR, string _satis_USD, string _satis_EUR, string _satis_GBP, string _satis_SAR)
        {
            char[] separator = new char[] { ',' };
            foreach (string str in kurmailler.Split(separator))
            {
                SmtpClient client = new SmtpClient
                {
                    Port = Convert.ToInt32(_MailServerPort),
                    Host = _MailServerSmtp,
                    EnableSsl = Convert.ToBoolean(true),
                    Timeout = Convert.ToInt32(zaman),
                    Credentials = new NetworkCredential(gondericikul, gondericisifre)
                };
                MailMessage message = new MailMessage();
                message.To.Add(str);
                message.From = new MailAddress(gondericikul, gondericibaslik);
                message.Subject = gondericibaslik;
                message.IsBodyHtml = true;
                string textArray1 = "<table border='0' cellpadding='6' cellspacing='5'><tr><td colspan='4' align='center' valign='middle' ><strong>";
                textArray1 += gondericibaslik;
                textArray1 += "  Tarih :  ";
                textArray1 += DateTime.Now.ToString();
                textArray1 += " </strong></td></tr><tr><td align='left' valign='middle' ><strong>Name</strong></td><td align='center' valign='middle'><strong>Code</strong></td><td align='center' valign='middle'><strong>Alış</strong></td><td align='center' valign='middle'><strong>Satış</strong></td></tr><tr><td>DOLAR Efektif </td><td>USD</td><td>";
                textArray1 += _alis_USD;
                textArray1 += " TL</td><td>";
                textArray1 += _satis_USD;
                textArray1 += " TL</td></tr><tr><td>EURO Efektif </td><td>EUR</td><td>";
                textArray1 += _alis_EUR;
                textArray1 += " TL</td><td>";
                textArray1 += _satis_EUR;
                textArray1 += " TL</td></tr><tr><td>INGILIZ STERLINI Efektif </td><td>GBP</td><td>";
                textArray1 += _alis_GBP;
                textArray1 += " TL</td><td>";
                textArray1 += _satis_GBP;
                textArray1 += " TL</td></tr><tr><td>SUUDI ARABISTAN RIYALI  Efektif </td><td>SAR</td><td>";
                textArray1 += _alis_SAR;
                textArray1 += " TL </td><td>";
                textArray1 += _satis_SAR;
                textArray1 += " TL</td></tr></table>";
                message.Body = string.Format(textArray1);
                client.Send(message);
                message.Dispose();
                client.Dispose();
            }


        }

        public class DovizKurlari
        {
            public string Kod { get; set; }
            public string Ad { get; set; }
            public string AlisFiyat { get; set; }
            public string SatisFiyat { get; set; }
        }
    }
}
