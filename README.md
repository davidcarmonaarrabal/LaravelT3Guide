# CÓMO HACER EL PROYECTO:

## 1. CREAMOS EL PROYECTO
Bien, empezamos a crear el proyecto, ya sabemos `laravel new examen`, seleccionando dentro `Laravel Breeze`, `Livewire (Volt Class API) with Alpine`, `PHPUnit` y `MySQL`.

Ahora **Cargamos nuestros datos de nuestra BBDD** en `.env`, en mi caso:

```
DB_HOST=127.0.0.1 
DB_PORT=3306 #(Puerto que usa MySQL)
DB_DATABASE=proyecto1 #(Nombre de la base de datos que usará el proyecto, normalmente igual que el nombre del proyecto)
DB_USERNAME=root 
DB_PASSWORD=david 
```

Bien, tras esto, cargamos las migraciones con `php artisan serve` y en otra terminal `php artisan migrate`, **seleccionando SÍ a crear la BBDD**.

## 2. CREAMOS EL MODELO Y MIGRACIONES
Bien, ahora vamos a crear 3 tablas, **Clientes**, **Transportistas** y **Pedidos**, con sus determinadas relaciones entre ellas, como **Clientes** y **Transportistas** en ***1:N*** con **Pedidos**, es decir:

* Un **Cliente** puede hacer varios **Pedidos**, pero este sólo pertenece a un **Cliente**.
* Lo mismo con los **Transportistas**.

En este caso, los **Clientes** serán **Users**, ya que ***Laravel*** nos facilita esta tabla como una de las principales para el login:

```
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->string('adress');
            $table->integer('telephone');
            $table->rememberToken();
            $table->timestamps();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
```

Luego, seguimos con **Transportistas**:

```
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('transportistas', function (Blueprint $table) {
            $table->id();
            $table->string('name_trans');
            $table->integer('telephone_trans');
            $table->string('vehicle_type');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('transportistas');
    }
};
```

Y por último, con **Pedidos**:

```
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('pedidos', function (Blueprint $table) {
            $table->id();
            $table->date('fecha_pedido');
            $table->date('fecha_entrega');
            $table->string('status');
            $table->unsignedBigInteger('user_id')->unsigned();
            $table->unsignedBigInteger('transportista_id')->unsigned();
            $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
            $table->foreign('transportista_id')->references('id')->on('transportistas')->onDelete('cascade');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('pedidos');
    }
};
```

Ahora, procedemos con los ***Modelos***, primero el de **Pedidos**:

```
class Pedidos extends Model
{
    use SoftDeletes, HasFactory;

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function transportista(): BelongsTo
    {
        return $this->belongsTo(Transportista::class);
    }
}
```

Añadimos ambas relaciones, ahora pasamos a **Users**:

```
class User extends Authenticatable
{
    use HasFactory, Notifiable, SoftDeletes;
    protected $fillable = [
        'name',
        'email',
        'password',
        'adress',
        'telephone',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }

    public function pedidos(): HasMany
    {
        return $this->hasMany(Pedido::class, 'user_id', 'id');
    }
}
```

```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\SoftDeletes;

class Transportista extends Model
{
    use SoftDeletes, HasFactory;

    public function pedidos(): HasMany
    {
        return $this->hasMany(Pedidos::class, 'transportista_id', 'id');
    }
}
```

Y con esto establecemos todos los modelos, 
