# Scripting en Unity

## Entrada de teclado & físicas

### Movimiento horizontal (izquierda-derecha):
```c#
float dirX = Input.GetAxis("Horizontal");
rb.velocity = new Vector2(dirX * moveSpeed, rb.velocity.y);
```

### Pulsación de una tecla concreta (indicando la tecla)
Comprobar que se pulsa una tecla en concreto (onKeyDown) y en su caso impulsamos 14 unidades:
```c#
if (Input.GetKeyDown("space")) {    // En lugar de "space" se puede usar KeyCode.Space
    GetComponent<Rigidbody2D>().velocity = new Vector2(0, 14);
}
```

### Pulsación de una tecla (indicando la acción)
Comprobar que se pulsa un botón (sin indicar la tecla exacta, sino la acción):
```c#
float jumpValue = Input.GetButtonDown("Jump"); // Entre 0 y 1
if (jumpValue) rb.velocity = new Vector2(0, jumpForce); // Falta comprobar que toca suelo
```

### Forma sencilla de comprobar si un cuerpo toca el suelo
```c#
// Es necesario crear una capa (layer) para asignarle
[SerializeField] private LayerMask jumpableGround;
...
private bool isGrounded() => Physics2D.BoxCast(
    coll.bounds.center,
    coll.bounds.size,
    0f,
    Vector2.down,
    .1f,
    jumpableGround
);
```

## Animaciones
Es útil declarar un enumerado con los tipos de animaciones que puedan producirse en nuestro juego. Por ejemplo:
```c#
private enum AnimationType { idle, running, jumping, falling }
```

Otro ejemplo con muchas más posibilidades (si tuviésemos un juego más complejo y con muchas animaciones):
```c#
public enum AnimationType
{
    die,
    hit,
    idle,
    attack,
    run,
    jump,
    fall,
    climb,
    land
}
```

El sistema que viene con Unity se llama [Mecanim](https://docs.unity3d.com/Manual/AnimationOverview.html). Podemos hacer uso de el enlazando pantallas en un Animator de Unity o simplemente manejar scripts.

### Forma I: utilizando el animator de Unity
```c#
private Animator animator;
...
private void UpdateAnimationState()
{
    AnimationType state; // Declaro una variable del tipo del enumerado anterior (del sencillo)

    if (dirX != 0f)
    {
        state = AnimationType.running;
        sprite.flipX = dirX < 0f; // Si es menor que 0 hace flip (voltea el sprite)
    }
    else
    {
        state = AnimationType.idle;
    }

    if (rb.velocity.y > .1f)
    {
        state = AnimationType.jumping;
    }
    else if (rb.velocity.y < -.1f)
    {
        state = AnimationType.falling;
    }

    // Haciendolo con el animator de Unity establecemos una variable del animator
    anim.SetInteger("state", (int)state); 
}

```

### Forma II: manual
Habría que variar lo anterior para que en lugar de anim.SetInteger(...) utilice [Animator.Play(...)](https://docs.unity3d.com/ScriptReference/Animator.Play.html).
```c#
private Animator animator;
...
// -1 default para la capa, 0 para que empiece sin ningún retardo
animator.Play("Idle", -1, 0f);
```


# Coleccionar items
El GameObject debe estar marcado con IsTrigger y un Tag (p. ej: "coin" o "moneda"). El callback para manejarlo se llama **OnTriggerEnter2D**:
```c#
private void OnTriggerEnter2D(Collider2D collision)
{
    // Detecta la colision del GameObject actual con otro objeto
    if (collision.gameObject.CompareTag("coin"))
    {
        // Destruye un objeto una vez recogido
        Destroy(collision.gameObject);
    }
}
```

Mismo ejemplo ampliado (pero que además guarda la moneda y muestra el texto en la GUI):
```c#
private int coins = 0;
[SerializeField] private Text textCoins;

...

private void OnTriggerEnter2D(Collider2D collision)
{
    // Detecta la colision del GameObject actual con otro objeto
    if (collision.gameObject.CompareTag("coin"))
    {
        // Destruye un objeto una vez recogido
        Destroy(collision.gameObject);

        // Podría aumentar el número de monedas y mostrar el texto en la pantalla
        coins++;
        textCoins.text = "Numero de monedas: " + coins;
    }
}
```
