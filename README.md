![Untitled](https://github.com/quocitspkt/testreadme/blob/main/r1.png)

-   Mở folder CMDScript

![Untitled](https://github.com/quocitspkt/testreadme/blob/main/r2.png)

-   Chạy file [DownloadPackageNative_ForWindows.sh](http://downloadpackagenative.sh) để tải Pagackage về. (Khi chạy filde .sh phải thoát Unity ra trước)

![Untitled](https://github.com/quocitspkt/testreadme/blob/main/r3.png)

-   Sau khi chạy Download xong Packages sẽ nằm trong folder UniVR/Assets/Packages

    ![send2.png](https://github.com/quocitspkt/testreadme/blob/main/r4.png)

-   Tiếp theo, chỉnh sửa lại đường dẫn trong file [ImportAssetPackge.sh](http://importassetpackge.sh) và chạy.

![send4.png](https://github.com/quocitspkt/testreadme/blob/main/r5.png)

-   '/c/Program Files/Unity/Hub/Editor/2020.3.16f1/Editor/Unity.exe': là đường dẫn trỏ tới file Unity.exe để thực thi Unity
-   ../../UniVR/: Là đường dẫn tới project, ở đây tên project là UniVR, nếu chạy Window thì giữ nguyên cũng được, còn chạy bằng MacOS thì đổi cho phù hợp với cấu trúc thư mục của MacOS
-   "G:\Unity\Project\UniVR_Send_4\UniVR\UniVR\Assets\Packages\\$pkname" : Là đường dẫn trỏ tới file .unitypackage
-   Đối với MacOS cũng đổi đường dẫn tới app Unity cho phù hợp với cấu trúc thư mục của MacOS
-   Tiếp theo, chỉnh sửa lại đường dẫn trong file **[ExportProjectAndroid.sh](http://ExportProjectAndroid.sh)**

![send5.png](https://github.com/quocitspkt/testreadme/blob/main/r6.png)

-   '/c/Program Files/Unity/Hub/Editor/2020.3.16f1/Editor/Unity.exe': là đường dẫn trỏ tới file Unity.exe để thực thi Unity
-   ../../UniVR/: Là đường dẫn tới project, ở đây tên project là UniVR, nếu chạy Window thì giữ nguyên cũng được, còn chạy bằng MacOS thì đổi cho phù hợp với cấu trúc thư mục của MacOS
-   Đối với MacOS cũng đổi đường dẫn tới app Unity cho phù hợp với cấu trúc thư mục của MacOS
-   **Làm tương tự như vậy cho file [ExportProjectIOS.sh](http://ExportProjectIOS.sh),ExportProjectAndroid**
-   Định nghĩa các thuộc tính để export project iOS theo ý mình trong file Assets/Editor/CustomUtilities.cs.

```
[MenuItem("Custom Utilities/Export project IOS Xcode")]
    static void ExportXcodeProject()
    {
        string accessKey = AppConfig.ACCESS_KEY;
        string accessSecret = AppConfig.ACCESS_SECRET;
        string bucket = AppConfig.BUCKET_NAME;

        string compName = "Sunshine TECH";
        string prodName = "UniVR-personal";
        string version = "0.4";
        int screenWidth = 640;
        int screenHeight = 480;
        bool fullScreen = true;
        EditorUserBuildSettings.SwitchActiveBuildTarget(BuildTarget.iOS);

        //config player setting 
        PlayerSettings.companyName = compName;
        PlayerSettings.productName = prodName;
        PlayerSettings.bundleVersion = version;

        //config resoloution
        PlayerSettings.defaultScreenWidth = screenWidth;
        PlayerSettings.defaultScreenHeight = screenHeight;
        PlayerSettings.defaultIsFullScreen = fullScreen;

        //Set default Icon
        Texture2D iconImage = (Texture2D)AssetDatabase.LoadMainAssetAtPath("Assets/TutorialInfo/Icons/UniversalIcon.png");
        PlayerSettings.SetIconsForTargetGroup(BuildTargetGroup.Unknown, new Texture2D[] { iconImage });


        EditorUserBuildSettings.symlinkLibraries = true;
        EditorUserBuildSettings.development = true;
        EditorUserBuildSettings.allowDebugging = true;

        List<string> scenes = new List<string>();
        //Define scenes
        //scenes.Add("Assets/DevResources/Scenes/RoomDemo.unity");


        using (AmazonS3Client s3Client = new AmazonS3Client(accessKey, accessSecret, Amazon.RegionEndpoint.APSoutheast1))
        {
            var fileTransferUtility = new TransferUtility(s3Client);
            Stream stream = fileTransferUtility.OpenStream(bucket, AppConfig.JSON_SCENEPATH);//(bucket, folderPath + "/" + "fileKey")
            using (var memoryStream = new MemoryStream())
            {
                stream.CopyTo(memoryStream);
                var response = memoryStream.ToArray();
                var strJson = System.Text.Encoding.Default.GetString(response);

                AppConfig.app = JsonUtility.FromJson<App>(strJson);
            }
        }
        foreach (var item in AppConfig.app.scenepath)
        {
            scenes.Add(item.scenepathname);
        }

        string savePath = Path.GetFullPath(".") + "\\iOSProject";
        BuildPipeline.BuildPlayer(scenes.ToArray(), savePath, BuildTarget.iOS, BuildOptions.None);
    }

```

-   Có thể định nghĩa các scenes cần build theo ý mình, ở đây mặc định sẽ build các scenes được định nghĩa trong file ConfigScenePath.json.
-   Sau khi Build xong, sản phẩm sẽ nằm ở thư mục UniVR/iOSProject
-   Export project Android cũng tương tự như iOS
