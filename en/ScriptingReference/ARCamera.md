# ARCamera

> Namespace: Ximmerse.RhinoX      
> public sealed class ARCamera : MonoBehaviour

![Logo](https://raw.githubusercontent.com/yinyuanqings/AIOSDK/gh-pages/img/Inspector/ARCamera.png ':size=450X400')

ARCamera is a virtual representation of HMD in virtual world. ARCamera provides the following functionalities in SDK:
- Update ARCamera Transform based on user real time movement.
- ARCamera component drives Tag Tracking module，which scans physical objects.
- Stereoscopic rendering.
- Cursor rendering. The cursor can be used to interact with Unity interactable targets, such as uGUI buttons, and provide interaction events.

!> ARCamera is a singleton and there should only be one instance in the scene. Developers can get a refference to ARCamera by using ARCamera.Instance.

### Public Members

#### TrackPosition
- `public bool TrackPosition { get; set; };// Default : true`

ARCamera updates its positon in real time based on user's physical movement.


#### TrackRotation
- `public bool TrackRotation { get; set; }; // Default : true`

AARCamera updates its orientation in real time based on user's physical movement.




#### IsObjectTrackingEnabled
- `public bool IsObjectTrackingEnabled { get; } `

If this value is true, it means that Object Tracking Engine is successfully initialized. `SetObjecTrackingProfile()` only works after `IsObjectTrackingEnabled == True` is true.

```bash
//Example code: how to loads a new tracking profile:
IEnumerator ChangeTrackingProfile ()
{
   while(ARCamera.Instance.IsObjectTrackingEnabled == false)
   {
      yield return null;//wait for object tracking enabled.
   }
   ARCamera.Instance.SetObjecTrackingProfile(newProfile: Resources.Load<ObjectTrackingProfile>("Cube+Beacons"));//loads new profile
}
```


#### EnableReticle
- `public bool EnableReticle { get; set; } ; // Default : true`

Reticle is a small round cursor rendered on screen, representing where the user is looking at. `Reticle Ray` is used to interact with GameObjects.
`EnableReticle` is used to disable or enable Reticle rendering. When `EnableReticle` is set to true, `Reticle Ray Generator` is enabled.

!> <b>`Reticle Ray` only works with GameObjects defined under `InteractMask` when `RenderRectile is` set to true, and `RxEventSystem` and `RxInputModule` are both existed in the scene. </b>


#### ReticleImage
- `public Texture2D ReticleImage { get; set; } // See : RhinoXGlobalSetting.DefaultRectileImage`

Reticle Image, which can be customized.
If `ReticleImage` is null, reticle will not be rendered, but raycast will still work.

```bash
    void ChangeReticleTexture()
    {
        ARCamera arCamera = ARCamera.Instance;
        arCamera.ReticleImage = Resources.Load<Texture2D>("MyReticle");
    }
```

#### ReticleInteractMask
- `public LayerMask ReticleInteractMask { get; set; } // Default : see : 1 << 5 (UI mask)`

When `EnableReticle` is set to true, developer can define interactable object layers. 

```bash
    void ChangeReticleTexture()
    {
        ARCamera arCamera = ARCamera.Instance;
        arCamera.InteractMask = LayerMask.GetMask("Default", "UI");
    }
```

#### TrackingProfile
- public ObjectTrackingProfile TrackingProfile { get; } // Default : null

Gets currently used Tracking Profile. Tracking Project is set through ARCamera Inspector. If changing Tracking Profile is desired, please use `SetObjecTrackingProfile()`.


#### InterPupilDistance
- `public float InterPupilDistance { get; set; } // Default : 0.062`

Pupillary distance is set to 62mm by default. Developers can customize this value.

```bash
    void ChangeIPD()
    {
        ARCamera arCamera = ARCamera.Instance;
        arCamera.InterPupilDistance = 0.060f;//change IPD to 60mm (default 62mm)
    }
```

### Public Methods


#### GetReticleRay()
- `public Ray GetReticleRay();`

Get a ray from `Ray Generator`.

```bash
    void GetGazeAt()
    {
        Ray ray = ARCamera.Instance.GetReticleRay();
        RaycastHit hitInfo;
        if(Physics.Raycast (ray, out hitInfo))
        {
            Debug.LogFormat("Gaze at : {0}", hitInfo.collider.name);
        }
    }
```

#### SetObjecTrackingProfile
- `public bool SetObjecTrackingProfile(ObjectTrackingProfile newProfile);`

When loading a new Tracking Profile, the previous Tracking Profile will be deleted from the memory.

```bash
//Example code: how to loads a new tracking profile:
IEnumerator ChangeTrackingProfile ()
{
   while(ARCamera.Instance.IsObjectTrackingEnabled == false)
   {
      yield return null;//wait for object tracking enabled.
   }
   ARCamera.Instance.SetObjecTrackingProfile(newProfile: Resources.Load<ObjectTrackingProfile>("Cube+Beacons"));//loads new profile
}
```

!> <b>Loading Tracking Profile is an asynchronous operation, so the operation is not finished immediately. However, the entire operation typically only 1 frame.</b>

#### RecenterTracking()
- `public void RecenterTracking();`

Recenter the ARCamera, setting localPosition and localRotation to zero.



### Static Members

#### Instance
- `public static ARCamera Instance { get; }`

ARCamera static refference.

```bash
    void Update()
    {
        //Print head trcked position and rotation per frame:
        ARCamera arCamera = ARCamera.Instance;
        Debug.LogFormat("Head position:{0}, rotation:{1}", arCamera.transform.localPosition, arCamera.transform.localEulerAngles);
    }
```


#### OnCreateStereoCameras
- `public static event System.Action<Camera LeftEye, Camera RightEye>` 

Developers can listen the events triggered by left and right eye cameras.

```bash
    void Start()
    {
        ARCamera.OnCreateStereoCameras += OnCreateStereoCameras;
    }

    /// <summary>
    /// Creates post image effect : blur on left and right eye.
    /// </summary>
    /// <param name="leftEye">Left eye.</param>
    /// <param name="rightEye">Right eye.</param>
    void OnCreateStereoCameras(Camera leftEye, Camera rightEye)
    {
        leftEye.gameObject.AddComponent<BlurImageEffect>();
        rightEye.gameObject.AddComponent<BlurImageEffect>();
        ARCamera.OnCreateStereoCameras -= OnCreateStereoCameras;
    }
```