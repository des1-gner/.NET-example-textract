# Amazon Textract OCR with .NET on EC2 (Amazon Linux 2023)

This guide shows how to set up Amazon Textract for OCR using .NET on an EC2 instance running Amazon Linux 2023.

## Prerequisites

1. Launch an EC2 instance with Amazon Linux 2023
2. Create an IAM role with the following permissions and attach it to your EC2 instance

### Required IAM Policy
Create an IAM role with this policy (replace `<your-bucket-name>` with your actual S3 bucket name):

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "textract:DetectDocumentText",
                "textract:AnalyzeDocument"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::<your-bucket-name>/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::<your-bucket-name>"
        }
    ]
}
```

## Installation Steps

### 1. Update system and install .NET 8 SDK
```bash
sudo yum update -y
sudo yum install dotnet-sdk-8.0 -y

# Verify installation
dotnet --version
```

### 2. Create .NET project
```bash
cd ~
mkdir textract-ocr-dotnet
cd textract-ocr-dotnet
dotnet new console
```

### 3. Add AWS SDK packages
```bash
dotnet add package AWSSDK.Textract
dotnet add package AWSSDK.S3
```

### 4. Replace Program.cs content
Replace the content of `Program.cs` with the following (replace `<your-bucket-name>`, `<your-region>`, and `<your-document-path>` with your actual values):

```csharp
using Amazon;
using Amazon.Textract;
using Amazon.Textract.Model;
using System;
using System.IO;
using System.Text.Json;
using System.Threading.Tasks;

namespace TextractOcrDotnet
{
    class Program
    {
        static async Task Main(string[] args)
        {
            try
            {
                // Create Textract client
                var textractClient = new AmazonTextractClient(RegionEndpoint.USEast1); // Change to your region
                
                // Set up document
                var document = new Document
                {
                    S3Object = new Amazon.Textract.Model.S3Object
                    {
                        Bucket = "<your-bucket-name>",
                        Name = "<your-document-path>" // e.g., "documents/resume.pdf"
                    }
                };
                
                Console.WriteLine("Calling Amazon Textract...");
                
                // For OCR (text extraction)
                var request = new DetectDocumentTextRequest
                {
                    Document = document
                };
                
                var response = await textractClient.DetectDocumentTextAsync(request);
                
                Console.WriteLine("Processing response...");
                
                // Process results
                foreach (var block in response.Blocks)
                {
                    if (block.BlockType == BlockType.LINE)
                    {
                        Console.WriteLine(block.Text);
                    }
                }
                
                // Save full result to file
                string jsonString = JsonSerializer.Serialize(response, new JsonSerializerOptions { WriteIndented = true });
                File.WriteAllText("output.json", jsonString);
                Console.WriteLine("Results saved to output.json");
            }
            catch (Exception e)
            {
                Console.WriteLine($"Error: {e.Message}");
                Console.WriteLine(e.StackTrace);
            }
        }
    }
}
```

### 5. Build and run
```bash
dotnet build
dotnet run
```

## Expected Output

The application will:
1. Extract text from your PDF document using Amazon Textract
2. Print each line of text to the console
3. Save the complete response to `output.json`

## Troubleshooting

- **Permission errors**: Ensure your EC2 instance has the correct IAM role attached
- **Region mismatch**: Make sure the region in your code matches your S3 bucket's region
- **File not found**: Verify the S3 bucket name and file path are correct
- **.NET not found**: Ensure .NET 8 SDK is properly installed

## Notes

- This example uses `DetectDocumentText` for basic OCR
- For form data extraction, use `AnalyzeDocument` with feature types
- For table extraction, use `AnalyzeDocument` with feature types
- The application uses IAM roles for authentication (recommended for EC2)
- .NET automatically discovers AWS credentials from the EC2 instance metadata
