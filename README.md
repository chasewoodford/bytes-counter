bytes-counter
=============

App that allows you to take a picture of the food you eat and tracks how many calories you consume.

**Resources:**
http://notanothercodeblog.blogspot.com/2011/02/google-goggles-api.html
http://social.msdn.microsoft.com/Forums/en-US/csharpgeneral/thread/755f3ab5-95df-4978-9ece-ee383e62be04/


### Project Outline

**Part 1:** 
Retrieve image data from source and turn into a bit array

```c#
BitArray bitArray  = new BitArray(File.ReadAllBytes("C:\\Image.jpg"));
```

**Part 2:** 
Post image to Google Goggles

```c#
namespace GoogleGoggles.NET
{
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Net;
 
    public class GoogleGoggles
    {
        // The POST body required to validate the CSSID.
        private static byte[] CssidPostBody = new byte[] { 34, 0, 98, 60, 10, 19, 34,
            2, 101, 110, 186, 211, 240, 59, 10, 8, 1, 16, 1, 40, 1, 48, 0, 56, 1, 18,
            29, 10, 9, 105, 80, 104, 111, 110, 101, 32, 79, 83, 18, 3, 52, 46, 49,
            26, 0, 34, 9, 105, 80, 104, 111, 110, 101, 51, 71, 83, 26, 2, 8, 2, 34,
            2, 8, 1  };
  
        // Bytes trailing the image byte array. Look at the next code snippet to see
        // where it is used in SendPhoto() method.
        private static byte[] TrailingBytes = new byte[] { 24, 75, 32, 1, 48, 0, 146,
            236, 244, 59, 9, 24, 0, 56, 198, 151, 220, 223, 247, 37, 34, 0  };
 
        // Generates a cssid.
        private static string Cssid
        {
            get
            {
                Random random = new Random((int)DateTime.Now.Ticks);
                return string.Format(
                    "{0}{1}",
                    random.Next().ToString("X8"),
                    random.Next().ToString("X8"));
            }
        }
 
        static void Main(string[] args)
        {
            string cssid = GoogleGoggles.Cssid;
            GoogleGoggles.ValidateCSSID(cssid);
 
            // See next code snippet for SendPhoto()
            //GoogleGoggles.SendPhoto(cssid, yourImageByteArray);
            Console.ReadLine();
        }
 
        // Validates the CSSID we just created, by POSTing it to Goggles.
        private static void ValidateCSSID(string cssid)
        {
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create(
                string.Format(
                "http://www.google.com/goggles/container_proto?cssid={0}",
                cssid));
 
            GoogleGoggles.AddHeaders(request);
            request.Method = "POST";
 
            using (Stream stream = request.GetRequestStream())
            {
                stream.Write(
                    GoogleGoggles.CssidPostBody,
                    0,
                    GoogleGoggles.CssidPostBody.Length);
 
                stream.Flush();
            }
 
            HttpWebResponse response = (HttpWebResponse)request.GetResponse();
        }
 
        private static void AddHeaders(HttpWebRequest request)
        {
            request.ContentType = "application/x-protobuffer";
            request.Headers["Pragma"] = "no-cache";
            request.KeepAlive = true;
        }
    }
}

// Conducts an image search by POSTing an image to Goggles, along with a valid CSSID.
public static void SendPhoto(string cssid, byte[] image)
{
    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(
        string.Format(
        "http://www.google.com/goggles/container_proto?cssid={0}",
        cssid));
 
    GoogleGoggles.AddHeaders(request);
    request.Method = "POST";
 
    // x = image size
    int x = image.Length;
    byte[] xVarint = GoogleGoggles.ToVarint32(x).ToArray<byte>();
 
    // a = x + 32
    byte[] aVarint = GoogleGoggles.ToVarint32(x + 32).ToArray<byte>();
 
    // b = x + 14
    byte[] bVarint = GoogleGoggles.ToVarint32(x + 14).ToArray<byte>();
 
    // c = x + 10
    byte[] cVarint = GoogleGoggles.ToVarint32(x + 10).ToArray<byte>();
 
    // 0A [a] 0A [b] 0A [c] 0A [x] [image bytes]
    using (Stream stream = request.GetRequestStream())
    {
        // 0x0A
        stream.Write(new byte[] { 10 }, 0, 1);
 
        // a
        stream.Write(aVarint, 0, aVarint.Length);
 
        // 0x0A
        stream.Write(new byte[] { 10 }, 0, 1);
 
        // b
        stream.Write(bVarint, 0, bVarint.Length);
 
        // 0x0A
        stream.Write(new byte[] { 10 }, 0, 1);
 
        // c
        stream.Write(cVarint, 0, cVarint.Length);
 
        // 0x0A
        stream.Write(new byte[] { 10 }, 0, 1);
 
        // x
        stream.Write(xVarint, 0, xVarint.Length);
 
        // Write image
        stream.Write(image, 0, image.Length);
 
        // Write trailing bytes
        stream.Write(
            GoogleGoggles.TrailingBytes,
            0,
            GoogleGoggles.TrailingBytes.Length);
 
        stream.Flush();
    }
 
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();
}
 
// Encodes an int32 into varint32.
public static IEnumerable<byte> ToVarint32(int value)
{
    int index = 0;
 
    while ((0x7F & value) != 0)
    {
        int i = (0x7F & value);
        if ((0x7F & (value >> 7)) != 0)
        {
            i += 128;
        }
 
        yield return ((byte)i);
        value = value >> 7;
        index++;
    }
}
```

**Part 3:**
Append search input with phrase like "calories

**Part 4:**
Scrape results to get an average caloric number for whatever is in the image

**Part 5:**
Return value to a database accessible by app or user to track calorie intake 