# Apache Beam y Dataflow

Apache Beam es un modelo unificado de código abierto para definir canalizaciones de procesamiento de datos en paralelo y por lotes. Con uno de los SDK de Beam de código abierto, crea un programa que define la canalización. Luego, la canalización es ejecutada por uno de los back-end de procesamiento distribuido compatibles con Beam, que incluyen Apache Flink, Apache Spark y Google Cloud Dataflow.

Beam es particularmente útil para tareas de procesamiento de datos embarazosamente paralelas, en las que el problema se puede descomponer en muchos paquetes de datos más pequeños que se pueden procesar de forma independiente y en paralelo. También puede utilizar Beam para tareas de extracción, transformación y carga (ETL) e integración de datos puros. Estas tareas son útiles para mover datos entre diferentes medios de almacenamiento y fuentes de datos, transformar datos a un formato más deseable o cargar datos en un nuevo sistema.

## Conceptos básicos
### Canalizaciones
Una canalización encapsula la serie completa de procesamientos que participan en la lectura de los datos de entrada, la transformación de esos datos y la escritura de los datos de salida. La fuente de entrada y el receptor de salida pueden ser del mismo tipo o pueden ser de tipos diferentes, lo que te permite convertir los datos de un formato en otro con facilidad. Los programas de Apache Beam comienzan por crear un objeto Pipeline y, luego, usan ese objeto como la base para crear los conjuntos de datos de canalización. Cada canalización representa un trabajo único y repetible.
### PCollection
Una PCollection representa un conjunto de datos de elementos múltiples que puede distribuirse y que actúa como los datos de la canalización. Las transformaciones de Apache Beam utilizan objetos PCollection como entradas y salidas para cada paso de tu canalización. Una PCollection puede contener un conjunto de datos de un tamaño fijo o un conjunto de datos no delimitado de una fuente de datos que se actualiza continuamente.
### Transformaciones
Una transformación representa una operación de procesamiento que transforma datos. Una transformación toma uno o más objetos PCollection como entrada, realiza una operación que especificas en cada elemento de esa colección y produce uno o más objetos PCollection como salida. Una transformación puede realizar casi cualquier tipo de operación de procesamiento, incluidos los cálculos matemáticos de datos, la conversión de datos de un formato a otro, la agrupación de datos, la lectura y escritura de datos, el filtro de datos para mostrar solo aquellos elementos que desees o la combinación de elementos de datos en valores únicos.
### ParDo
ParDo es la operación de procesamiento paralelo central de los SDK de Apache Beam, que invoca una función especificada por el usuario en cada uno de los elementos del objeto PCollection de entrada. ParDo recopila los elementos de salida en un objeto PCollection de salida. La transformación ParDo procesa los elementos de forma independiente y también puede hacerlo en paralelo.
### E/S de canalización
Los conectores de E/S de Apache Beam te permite leer datos en tu canalización y escribir datos de salida desde tu canalización. Un conector de E/S consta de una fuente y un receptor. Todas las fuentes y receptores de Apache Beam son transformaciones que permiten que las canalizaciones funcionen con datos de varios formatos de almacenamiento de datos diferentes. También puedes escribir un conector de E/S personalizado.
### Agregación
La agregación es el proceso de procesar algunos valores de varios elementos de entrada. El principal patrón de procesamiento para agregación en Apache Beam es agrupar todos los elementos con una clave y ventana comunes. Luego, combina cada grupo de elementos con una operación asociativa y conmutativa.
Funciones definidas por el usuario (UDF)
Algunas operaciones de Apache Beam permiten ejecutar código definido por el usuario para configurar la transformación. Para ParDo, el código definido por el usuario especifica la operación que se aplica a cada elemento, y para Combine, especifica cómo se deben combinar los valores. Una canalización puede contener UDF escritos en un lenguaje diferente del lenguaje del ejecutor. Una canalización también puede contener UDF escritos en varios lenguajes.
### Ejecutor
Los ejecutores son software que acepta una canalización y la ejecuta. La mayoría de los ejecutores son traductores o adaptadores para sistemas de procesamiento de macrodatos masivamente paralelos. Existen otros ejecutores diseñados para las pruebas locales y la depuración.

Apache beam tiene algunos ejecutores principales:
- Direct Runner
- Spark Runner
- Flink Runner
- Dataflow Runner


## Codigo
En python es usado el pickling para realizar todo el serializado de la clases, de esta manera se genera un archivo de texto donde va toda la información del proceso, de esta manera se genera algo "congelado" para ser usado siempre que sea necesario.

### Que puedes hacer con pickle?
Pickling es util para aplicaciones donde necesitas un grado de persistencia en tus datos. Los datos del estado de tu programa pueden ser guardados en disco, asi tu puedes continuar trabajando en ello tiempo despues. Tambien puede ser usado para enviar datos en TCP o una connecxion socket, o almacenar objetos python en la base de datos. Pickle es muy util cuando estas trabajando con algoritmos de machine learning, donde tu quieres guardarlos para ser usados para predicciones tiempo despues, sin necesidad de reescribir todo o volver a entrenar el modelo otra vez.

## PTransform Code

```python
class WriteToBigTable(beam.PTransform):
  """ A transform to write to the Bigtable Table.
  A PTransform that write a list of `DirectRow` into the Bigtable Table
  """
  def __init__(self, project_id=None, instance_id=None, table_id=None):
    """ The PTransform to access the Bigtable Write connector
    Args:
      project_id(str): GCP Project of to write the Rows
      instance_id(str): GCP Instance to write the Rows
      table_id(str): GCP Table to write the `DirectRows`
    """
    super(WriteToBigTable, self).__init__()
    self.beam_options = {
        'project_id': project_id,
        'instance_id': instance_id,
        'table_id': table_id
    }

  def expand(self, pvalue):
    beam_options = self.beam_options
    return (
        pvalue
        | beam.ParDo(
            _BigTableWriteFn(
                beam_options['project_id'],
                beam_options['instance_id'],
                beam_options['table_id'])))
```

## DoFn Code
Bigtable Writer DoFn Code.
```python
class _BigTableWriteFn(beam.DoFn):
  """ Creates the connector can call and add_row to the batcher using each
  row in beam pipe line
  Args:
    project_id(str): GCP Project ID
    instance_id(str): GCP Instance ID
    table_id(str): GCP Table ID
  """
  def __init__(self, project_id, instance_id, table_id):
    """ Constructor of the Write connector of Bigtable
    Args:
      project_id(str): GCP Project of to write the Rows
      instance_id(str): GCP Instance to write the Rows
      table_id(str): GCP Table to write the `DirectRows`
    """
    super(_BigTableWriteFn, self).__init__()
    self.beam_options = {
        'project_id': project_id,
        'instance_id': instance_id,
        'table_id': table_id
    }
    self.table = None
    self.batcher = None
    self.written = Metrics.counter(self.__class__, 'Written Row')

  def __getstate__(self):
    return self.beam_options

  def __setstate__(self, options):
    self.beam_options = options
    self.table = None
    self.batcher = None
    self.written = Metrics.counter(self.__class__, 'Written Row')

  def start_bundle(self):
    if self.table is None:
      client = Client(project=self.beam_options['project_id'])
      instance = client.instance(self.beam_options['instance_id'])
      self.table = instance.table(self.beam_options['table_id'])
    self.batcher = self.table.mutations_batcher()

  def process(self, row):
    self.written.inc()
    # You need to set the timestamp in the cells in this row object,
    # when we do a retry we will mutating the same object, but, with this
    # we are going to set our cell with new values.
    # Example:
    # direct_row.set_cell('cf1',
    #                     'field1',
    #                     'value1',
    #                     timestamp=datetime.datetime.now())
    self.batcher.mutate(row)

  def finish_bundle(self):
    self.batcher.flush()
    self.batcher = None

  def display_data(self):
    return {
        'projectId': DisplayDataItem(
            self.beam_options['project_id'], label='Bigtable Project Id'),
        'instanceId': DisplayDataItem(
            self.beam_options['instance_id'], label='Bigtable Instance Id'),
        'tableId': DisplayDataItem(
            self.beam_options['table_id'], label='Bigtable Table Id')
    }
```
## BoundedSource Code
Los BoundedSource necesitan las funciones read, split, get_range_tracker y estimate_size.

**read**: Esta funcion genera los resultados de la lectura, usando el `range_tracker` para esto, tienes un `start_position` y un `end_position` para leer la informacion dentro de estos limites.
**split**: sirve para divir los paquetes, este puede ser usado N cantidad de veces sobre un mismo paquete. Sirve para distribuir la informacion a ser procesada.
**get_range_tracker**: Esta funcion sirve para definir el inicio y el fin de cada paquete, el resultado es el insertado en la funcion read.
**estimate_size**: Esta funcion es usada para estimar el tamaño del paquete de datos.
```python
class _CreateSource(iobase.BoundedSource):
  """Internal source that is used by Create()"""
  def __init__(self, serialized_values, coder):
    self._coder = coder
    self._serialized_values = []
    self._total_size = 0
    self._serialized_values = serialized_values
    self._total_size = sum(map(len, self._serialized_values))

  def read(self, range_tracker):
    start_position = range_tracker.start_position()
    current_position = start_position

    def split_points_unclaimed(stop_position):
      if current_position >= stop_position:
        return 0
      return stop_position - current_position - 1

    range_tracker.set_split_points_unclaimed_callback(split_points_unclaimed)
    element_iter = iter(self._serialized_values[start_position:])
    for i in range(start_position, range_tracker.stop_position()):
      if not range_tracker.try_claim(i):
        return
      current_position = i
      yield self._coder.decode(next(element_iter))

  def split(self, desired_bundle_size, start_position=None, stop_position=None):
    if len(self._serialized_values) < 2:
      yield iobase.SourceBundle(
          weight=0,
          source=self,
          start_position=0,
          stop_position=len(self._serialized_values))
    else:
      if start_position is None:
        start_position = 0
      if stop_position is None:
        stop_position = len(self._serialized_values)
      avg_size_per_value = self._total_size // len(self._serialized_values)
      num_values_per_split = max(
          int(desired_bundle_size // avg_size_per_value), 1)
      start = start_position
      while start < stop_position:
        end = min(start + num_values_per_split, stop_position)
        remaining = stop_position - end
        # Avoid having a too small bundle at the end.
        if remaining < (num_values_per_split // 4):
          end = stop_position
        sub_source = Create._create_source(
            self._serialized_values[start:end], self._coder)
        yield iobase.SourceBundle(
            weight=(end - start),
            source=sub_source,
            start_position=0,
            stop_position=(end - start))
        start = end

  def get_range_tracker(self, start_position, stop_position):
    if start_position is None:
      start_position = 0
    if stop_position is None:
      stop_position = len(self._serialized_values)
    from apache_beam import io
    return io.OffsetRangeTracker(start_position, stop_position)

  def estimate_size(self):
    return self._total_size
```