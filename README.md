using UnityEngine;
using UnityEngine.UI;

[RequireComponent(typeof(CharacterController))]
public class MobileTitanfallMovement : MonoBehaviour
{
    [Header("Movement")]
    public float moveSpeed = 8f;
    public float gravity = -9.81f;
    public float jumpForce = 6f;
    public int maxJumps = 2;

    [Header("Slide")]
    public float slideDuration = 0.8f;
    public float slideSpeed = 12f;

    [Header("Wall Run")]
    public float wallRunGravity = -2f;

    [Header("UI Controls")]
    public Joystick joystick;
    public Button jumpButton;
    public Button slideButton;

    private CharacterController controller;
    private Vector3 velocity;
    private int jumpCount = 0;
    private bool isSliding = false;
    private float slideTimer = 0f;
    private bool isWallRunning = false;

    void Start()
    {
        controller = GetComponent<CharacterController>();
        jumpButton.onClick.AddListener(MobileJump);
        slideButton.onClick.AddListener(StartSlide);
    }

    void Update()
    {
        HandleMovement();
        HandleSliding();
        HandleWallRun();
    }

    void HandleMovement()
    {
        // Ground check
        if (controller.isGrounded && velocity.y < 0)
        {
            velocity.y = -2f;
            jumpCount = 0;
        }

        // Joystick input
        Vector3 move = transform.right * joystick.Horizontal + transform.forward * joystick.Vertical;
        controller.Move(move * moveSpeed * Time.deltaTime);

        // Gravity
        if (!isWallRunning)
            velocity.y += gravity * Time.deltaTime;

        controller.Move(velocity * Time.deltaTime);
    }

    void MobileJump()
    {
        if (jumpCount < maxJumps)
        {
            velocity.y = Mathf.Sqrt(jumpForce * -2f * gravity);
            jumpCount++;
            isWallRunning = false;
        }
    }

    void StartSlide()
    {
        if (controller.isGrounded && !isSliding)
        {
            isSliding = true;
            slideTimer = slideDuration;
        }
    }

    void HandleSliding()
    {
        if (isSliding)
        {
            controller.Move(transform.forward * slideSpeed * Time.deltaTime);
            slideTimer -= Time.deltaTime;
            if (slideTimer <= 0f) isSliding = false;
        }
    }

    void HandleWallRun()
    {
        RaycastHit hit;
        bool wallRight = Physics.Raycast(transform.position, transform.right, out hit, 1f);
        bool wallLeft = Physics.Raycast(transform.position, -transform.right, out hit, 1f);

        if (!controller.isGrounded && (wallRight || wallLeft) && joystick.Vertical > 0.1f)
        {
            isWallRunning = true;
            velocity.y = wallRunGravity;
        }
        else
        {
            isWallRunning = false;
        }
    }
}
