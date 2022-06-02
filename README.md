using UnityEngine;
using UnityEngine.InputSystem;
public class CHarecter_MOvement : MonoBehaviour
{
     int iswalkHarsh;
    [SerializeField] private int isRunningHarsh;
    [SerializeField] private CharacterController characterController;
    [SerializeField] private @Player_InputAcio inputActions;
    [SerializeField] private Vector3 MovementCapture;
    [SerializeField] private Vector3 MovementInput;
    [SerializeField] public float speed;
    [SerializeField] private bool MovementActive;
    [SerializeField] private bool MovementActive_x;
    [SerializeField] private bool MovementActive_y;
    [SerializeField] private bool iswalking;
    [SerializeField] private bool isrunning;
    [SerializeField] private Animator animator;
    [SerializeField] public float smooth_speed;
    [SerializeField] private float smooth_Damp_Speed;
    [SerializeField] public float Shift_speed =10;
    [SerializeField] Transform cam;
    [SerializeField] float VerticalVelocity;
    [SerializeField] float gravity = 20f;
    [SerializeField] float jumpSpeed = 10.0f;
    [SerializeField] bool IsJump ;
    [SerializeField] float initialJumpVelocity;
    [SerializeField] float maxJumpHieght;
    [SerializeField] float maxJumpTime;
    [SerializeField] Transform GroundCheck;
    [SerializeField] float groundDistance = 0.4f;
    [SerializeField] LayerMask groundMask;
    [SerializeField] bool IsGrounded;
    [SerializeField] float _Gravity = -9.8f;
    [SerializeField] float GroundedGravity = -0.05f;
    [SerializeField] Vector3 velocity;


    [Header("Jump")]
    [SerializeField] float Jump_Distance;
    [SerializeField] float _jumpSpeed;
    [SerializeField] Transform OrigineRayCast;
    [SerializeField] float distance_IsWall;

    void Awake()
    {
        //IsGrounded = Physics.CheckSphere(GroundCheck.position, groundDistance, groundMask);
        //Calling @Player_InputActio scritps
        inputActions = new @Player_InputAcio();
        animator = GetComponent<Animator>();

        IsJump = Input.GetKeyDown(KeyCode.Space);
        //calling charecterController
        characterController = GetComponent<CharacterController>();
        inputActions.Player_INPUT.move.started += OnMove;
        inputActions.Player_INPUT.move.performed += OnMove;
        inputActions.Player_INPUT.move.canceled += OnMove;
    }
    void OnMove(InputAction.CallbackContext context)
    {
        // Debug.Log(context.ReadValue<Vector2>());
        MovementCapture = context.ReadValue<Vector2>();
        MovementInput.x = MovementCapture.x;
        MovementInput.z = MovementCapture.y;

        MovementActive = MovementInput.z != 0 || MovementInput.x != 0;
    }

    void Update()
    {
        Amimation();
        Movement();
        Gravity();
        climbing_System();
    }
    void climbing_System()
    {
        RaycastHit HitInfoIsWall;
        bool isWall = Physics.Raycast(OrigineRayCast.position,OrigineRayCast.forward,out HitInfoIsWall, distance_IsWall,groundMask);




    }

        
    void Gravity()
    {
         IsGrounded = Physics.CheckSphere(GroundCheck.position,groundDistance,groundMask);

        if (IsGrounded && velocity.y < 10f)
        {
            velocity.y = -2f;
            animator.SetBool("IsFalling",false);
        }
        else 
        {
            animator.SetBool("IsFalling", true);
        }

        if (!IsGrounded) {
            velocity.y -= gravity * Time.deltaTime;
            characterController.Move(velocity * Time.deltaTime);
        }
        if (Input.GetKeyDown(KeyCode.Space))
        {
            IsGrounded = true;
            Vector3 MoveUp = Vector3.up *jumpSpeed * Time.deltaTime;
            characterController.Move(MoveUp * Time.deltaTime);

        }
        else
        {
            IsGrounded = false;
        }
    }
    void Movement()
    {
        
         Vector3 MovementDirection = new Vector3(MovementInput.x, 0f, MovementInput.z);
        if (MovementDirection.magnitude >= 0.1f)
        {
            float tragetAngle = Mathf.Atan2(MovementInput.x, MovementInput.z) * Mathf.Rad2Deg + cam.eulerAngles.y;
            float Angle = Mathf.SmoothDamp(transform.eulerAngles.y, tragetAngle, ref smooth_Damp_Speed, smooth_speed);

            Vector3 MoveDirection = Quaternion.Euler(0f, Angle, 0f) * Vector3.forward;
            transform.rotation = Quaternion.Euler(0f, tragetAngle, 0f);
            characterController.Move(MoveDirection.normalized * speed * Time.deltaTime);
            if (Input.GetKey(KeyCode.LeftShift))
            {
                characterController.Move(MoveDirection.normalized * speed * Shift_speed * Time.deltaTime);
            }
        }

    }
    void Amimation()
    {

        iswalking = animator.GetBool("IsWalking");
        isrunning = animator.GetBool("IsRunning");
        bool isFalling = animator.GetBool("IsFalling");
        bool Isr = Input.GetKey(KeyCode.LeftShift);
        bool walk =Input.GetKey(KeyCode.W);

        if (MovementActive  &&!IsGrounded)
        {
            animator.SetBool("IsWalking", true);
        }
        else if (!MovementActive  && !IsGrounded)
        {
            animator.SetBool("IsWalking", false);
        }
        if ( (MovementActive && Isr)&&  !isrunning  && !IsGrounded)
        {
            animator.SetBool("IsRunning", true);
        }
        else if (!Isr && isrunning && !IsGrounded)
        {
            animator.SetBool("IsRunning", false);
        }
        else
        {
          //  Debug.Log("In Idle");
        }
    }

  
    void OnEnable()
    {
        inputActions.Enable();
    }
    void OnDisable()
    {
        inputActions.Disable();
    }
}
