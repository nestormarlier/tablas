from django.db import models
from django.contrib.auth.models import User
import datetime

from django.forms import model_to_dict


class FichaTecnica(models.Model):
    #ficha_tecnica_id = models.AutoField(verbose_name='Ficha Técnica Id', primary_key=True)
    nombre = models.CharField(max_length=50)
    active = models.BooleanField(
        verbose_name='Ficha Técnica Activo', default=True)

    created = models.DateTimeField(
        verbose_name='Fecha de alta', auto_now=True, db_column='creado')
    modified = models.DateTimeField(
        verbose_name='Fecha modificado', auto_now=True, db_column='modificado')

    user = models.ForeignKey(User, on_delete=models.CASCADE)
    # EN NULO SI SIGUE ACTIVO
    delete = models.DateField(verbose_name='Fecha baja', null=True, blank=True)

    def __str__(self):
        return self.nombre

    class Meta:
        verbose_name = 'Ficha Técnica'
        verbose_name_plural = 'Fichas Técnicas'
        db_table = 'ficha_tecnica'
        ordering = ['id']

class Categoria(models.Model):
    #categoria_id = models.IntegerField(verbose_name='Código de categoria', primary_key=True)
    nombre = models.CharField(max_length=30, unique=True)
    id_jefe = models.IntegerField(verbose_name='Identificación Jefe')

    created = models.DateTimeField(
        verbose_name='Fecha de alta', auto_now=True, db_column='creado')
    modified = models.DateTimeField(
        verbose_name='Fecha modificado', auto_now=True, db_column='modificado')

    def __str__(self):
        return self.nombre

    class Meta:
        db_table = 'categoria'
        verbose_name = 'Categoría'
        verbose_name_plural = 'Categorías'

    # def toJSON(self):
    #     # item = {'categoria_id': self.categoria_id, 'nombre': self.nombre, 'id_jede':self.id_jefe}
    #     item = model_to_dict(self)  # me permite conevertir mi entidad en diccionario , se pueden excludes ciertos
    #     # parametros
    #     return item

class Operario(models.Model):
    legajo = models.IntegerField(primary_key=True)
    nombre = models.CharField(max_length=20)
    apellido = models.CharField(max_length=20)
    categoria = models.ForeignKey(Categoria, on_delete=models.CASCADE)
    created = models.DateTimeField(
        verbose_name='Fecha de alta', auto_now=True, db_column='creado')
    modified = models.DateTimeField(
        verbose_name='Fecha modificado', auto_now=True, db_column='modificado')
    activo = models.BooleanField(verbose_name='Operario Activo')
    delete = models.DateField(verbose_name='Fecha baja', null=True,
                              blank=True)  # PUEDE ALMACENAR VALORES EN NULO SI SIGUE ACTIVO

    def __str__(self):
        return str(self.legajo) + ' - ' + self.nombre + ' - ' + self.apellido + ' - ' + str(self.categoria)

    class Meta:
        db_table = 'operario'
        ordering = ['legajo']

class Impresora(models.Model):
    impresora_id = models.IntegerField(primary_key=True)
    nombre = models.CharField(max_length=30)

    created = models.DateTimeField(
        verbose_name='Fecha de alta', auto_now=True, db_column='creado')
    modified = models.DateTimeField(
        verbose_name='Fecha modificado', auto_now=True, db_column='modificado')

    activo = models.BooleanField(verbose_name='Impresora Activa', default=True)
    # EN NULO SI SIGUE ACTIVO
    delete = models.DateField(verbose_name='Fecha baja', null=True, blank=True)

    def __str__(self):
        return "%s" % self.nombre

    class Meta:
        db_table = 'impresora'
        ordering = ['impresora_id']

class Parada(models.Model):
    NOMBRE_CHOICES = [
        ('TINTAS', 'TINTAS'),
        ('MONTAJE', 'MONTAJE'),
        ('DEPÓSITO', 'DEPÓSITO'),
        ('MANTENIMIENTO', 'MANTENIMIENTO'),
        ('PROGRAMACIÓN', 'PROGRAMACIÓN'),
        ('SUPERVICIÓN', 'SUPERVICIÓN'),
        ('PRODUCCIÓN', 'PRODUCCIÓN'),
        ('OPERATIVO', 'OPERATIVO'),
        ('CALIDAD', 'CALIDAD')
    ]
    #parada_id = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=40, unique=True)

    created = models.DateTimeField(
        verbose_name='Fecha de alta', auto_now=True, db_column='creado')
    modified = models.DateTimeField(
        verbose_name='Fecha modificado', auto_now=True, db_column='modificado')

    user = models.ForeignKey(User, on_delete=models.CASCADE)

    active = models.BooleanField(
        verbose_name='Ficha Técnica Activo', default=True)
    sector_asignado = models.CharField(max_length=30, choices=NOMBRE_CHOICES)
    # EN NULO SI SIGUE ACTIVO
    delete = models.DateField(verbose_name='Fecha baja', null=True, blank=True)

    def __str__(self):
        return self.nombre

    class Meta:
        db_table = 'parada'
        ordering = ['id']

class CambioMecanico(models.Model):
    #cambio_id = models.AutoField(primary_key=True)
    create = models.DateTimeField(
        verbose_name='Fecha creación', null=True, blank=True, default=datetime.datetime.now)
    modified = models.DateTimeField(
        verbose_name='Fecha modificado', auto_now=True, db_column='modificado')
    parada = models.ForeignKey(
        Parada, verbose_name='Parada', on_delete=models.CASCADE, null=True, blank=True)
    fecha_fin = models.DateTimeField(verbose_name='Fin cambio')

    class Meta:
        verbose_name = 'Cambio mecánico'
        verbose_name_plural = 'Cambios Mecánicos'
        db_table = 'cambio_mecanico'

    def __str__(self):
        return str(self.parada)

class Setup(models.Model):
    #setup_id = models.AutoField(primary_key=True)
    create = models.DateTimeField(
        verbose_name='Fecha creación', null=True, blank=True, default=datetime.datetime.now)
    parada = models.ForeignKey(
        Parada, verbose_name='Parada', on_delete=models.CASCADE, null=True, blank=True)
    fecha_fin = models.DateTimeField(verbose_name='Fin Setup')

    class Meta:
        verbose_name = 'Setup'
        verbose_name_plural = 'Setups'
        db_table = 'setup'

    def __str__(self):
        return str(self.parada)

class Produccion(models.Model):
    #produccion_id = models.AutoField(primary_key=True)
    create = models.DateTimeField(
        verbose_name='Fecha de creación', null=True, blank=True, default=datetime.datetime.now)
    parada = models.ForeignKey(
        Parada, verbose_name='Parada', on_delete=models.CASCADE, null=True, blank=True)
    fecha_fin = models.DateTimeField(verbose_name='Fin producción')

    class Meta:
        verbose_name = 'Producción'
        verbose_name_plural = 'Producciones'
        db_table = 'produccion'

    def __str__(self):
        return str(self.parada)

class ParteImpresion(models.Model):
    #parte_id = models.AutoField(verbose_name='Orden impresión', primary_key=True)
    maquinista = models.ForeignKey(Operario, verbose_name='Maquinista', on_delete=models.DO_NOTHING,
                                   related_name='maquinista', db_column='maquinista')
    supervisor = models.ForeignKey(Operario, verbose_name='Supervisor', on_delete=models.DO_NOTHING,
                                   related_name='supervisor', db_column='supervisor')
    ayudante1ero = models.ForeignKey(Operario, verbose_name='Primer ayudante', on_delete=models.DO_NOTHING,
                                     related_name='ayudante1er', db_column='ayudante1er')
    ayudante2do = models.ForeignKey(Operario, verbose_name='Segundo ayudante', on_delete=models.DO_NOTHING,
                                    related_name='ayudante2do', db_column='ayudante2do')
    fichaTecnica = models.ForeignKey(FichaTecnica, on_delete=models.CASCADE)
    impresora = models.ForeignKey(Impresora, on_delete=models.CASCADE)
    create = models.DateField(verbose_name='Fecha creación', auto_now=True)
    cambio = models.ManyToManyField(CambioMecanico, verbose_name="Cambio Mecánico", related_name='cambio',
                                    db_column='cambio_id', blank=True)
    setup = models.ManyToManyField(Setup, verbose_name='RM AC AP', related_name='setup', db_column='setup',
                                   blank=True)
    produccion = models.ManyToManyField(Produccion, verbose_name='Producción', related_name='produccion',
                                        db_column='produccion', blank=True)
    metros_registro = models.IntegerField(verbose_name='Metros registro')
    kg_registro = models.IntegerField(verbose_name='Kg. registro')
    metros = models.IntegerField(verbose_name='Metros producidos')
    kg = models.IntegerField(verbose_name='Kg producidos')

    class Meta:
        verbose_name = 'Parte de impresión'
        verbose_name_plural = 'Partes de impresión'
        db_table = 'parte_impresion'

    def __str__(self):
        return str(self.id)