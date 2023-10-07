This is going to be a short explanation of the FIM created in Josh Madakors [video](https://www.youtube.com/watch?v=WJODYmk4ys8) as it is not my code but was touched up in some areas where I saw improvement.

The script starts by giving the user 2 options:

**1) Creating a Baseline File**
**2) Monitoring the Folder**

Creating the baseline file tells the monitor what the file path of the folder is with all of the child items it contains. In this example it was the .\files folder which contained a few text files that were used to test.
If this file were to already to exist, the functionality is built so the original baselines.txt is deleted and replaced with the new one. It is stored in the format of (file path|hash) so later on when the file is used it can 
be split on the "|" character and stored in a dictionary with the path being the key and hash being the value. 
```
if($response -eq '1'){
    EraseBaseline

    $files = Get-ChildItem -Path .\files
    
    foreach($f in $files) {
        $hash = CalculateFileHash $f.FullName
        "$($hash.path)|$($hash.Hash)" | Out-File -FilePath.\baseline.txt -Append
        Write-Host "Added $($hash.Path) to baseline."
  }
}
```

Monitoring the files will have three main usecases. It will watch for new, changed and deleted files. If the user chooses this option the dictionary mentioned above will be created and then a while loop is initiated. This
is technically an endless loop as it will run until stopped every second to monitor the files. In the while loop, a foreach function is used to grab each text file in the files folder. 

If the file path equals null (which basically means it does not exist) it will write to the terminal ```Write-Host "$($hash.Path) has been created!"```. The ```$hash.Path``` comes from a function made above which calculates 
and returns the hash of a file using SHA512. With this returned hash, we call that function for each item in the files folder using the ```.FullName``` property to store the hash and file path in the ```$hash``` variable.

If the file path is found but the hash is not the same ```$fileHashDictionary[$hash.Path] -eq $hash.Hash```, this will write to the terminal that the file path has changed.

And if the file path was previously in the baselines.txt file but is no longer found in it, ```Write-Host "$($key) has been deleted!"``` will be written to the terminal. 

Doing these has gotten me more interested in learning Powershell on my own and I finally think I am ready to attempt to build something without
being spoon fed. In transparency, I attempted to create a much larger scope file monitor previously which would've included a gui and email notifications, but the tutorial included some steps which are no longer achievable due to the
age of the video. I was able to send alerts to email but integrating it to the file monitor contained issues with SecureString and PSCredential objects and I am not experienced enough to debug this yet. Although, I did learn many different
packages and different ways to interact with Powershell such as MimeKit and MailKit along with how to get their dependencies. 

Below is the code created by Josh, with a small revision to where the user enters "1" it will print the files added to the terminal. From here I will plan to research some ideas to make with Powershell scripts and attempt them on
my own to build my understanding with Powershell. While I do understand this project was essentially just copying Josh's monitor (I wrote it within VS Code but nearly verbatum to his), I think learning and displaying this as something I can go back to for reference will be very valuable 
in the learning process. This is why I want to give full credit to him for this monitor. 

```
Write-Host "What would you like to do?"
Write-Host "1) Collect new baseline?"
Write-Host "2) Begin monitoring files with saved baseline?"

$response = Read-Host -Prompt "Please enter '1' or '2'"

Write-Host "User entered $($response)."

function CalculateFileHash($filepath) {
    $filehash = Get-FileHash -Path $filepath -Algorithm SHA512
    return $filehash
}

function EraseBaseline() {
    $baselineExist = Test-Path -Path .\baseline.txt
    if($baselineExist){
        Remove-Item -Path .\baseline.txt
    }
}

if($response -eq '1'){
    EraseBaseline

    $files = Get-ChildItem -Path .\files
    
    foreach($f in $files) {
        $hash = CalculateFileHash $f.FullName
        "$($hash.path)|$($hash.Hash)" | Out-File -FilePath.\baseline.txt -Append
        Write-Host "Added $($hash.Path) to baseline."
    }
} elseif ($response -eq '2') {
    $fileHashDictionary = @{}

    $filePathsAndHashes = Get-Content -Path .\baseline.txt
    foreach($f in $filePathsAndHashes) {
        $fileHashDictionary.Add($f.Split("|")[0],$f.Split("|")[1])
        
    }

    while ($true) {
        Start-Sleep -Seconds 1

        $files = Get-ChildItem -Path .\files
    
        foreach($f in $files) {
            $hash = CalculateFileHash $f.FullName

            if($null -eq $fileHashDictionary[$hash.Path]) {
                Write-Host "$($hash.Path) has been created!" -ForegroundColor Green
                #"$($hash.path)|$($hash.Hash)" | Out-File -FilePath.\baseline.txt -Append

            } else {
                if($fileHashDictionary[$hash.Path] -eq $hash.Hash) {

                } else {
                    Write-Host "$($hash.path) has changed!" -ForegroundColor Red
                }
            }
        }
        foreach($key in $fileHashDictionary.Keys) {
            $baselineFileStillExists = Test-Path -Path $key
            if(-Not $baselineFileStillExists) {
                Write-Host "$($key) has been deleted!" -ForegroundColor Blue
            }
        }
    }
}
```
