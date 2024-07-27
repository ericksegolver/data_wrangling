# # Introducción
# 
# Instacart es una plataforma de entregas de comestibles donde la clientela puede registrar un pedido y hacer que se lo entreguen, similar a Uber Eats y Door Dash.
# El conjunto de datos que te hemos proporcionado tiene modificaciones del original. Redujimos el tamaño del conjunto para que tus cálculos se hicieran más rápido e introdujimos valores ausentes y duplicados. Tuvimos cuidado de conservar las distribuciones de los datos originales cuando hicimos los cambios.
# 
# Debes completar tres pasos. Para cada uno de ellos, escribe una breve introducción que refleje con claridad cómo pretendes resolver cada paso, y escribe párrafos explicatorios que justifiquen tus decisiones al tiempo que avanzas en tu solución.  También escribe una conclusión que resuma tus hallazgos y elecciones.
# 

# ## Diccionario de datos
# 
# Hay cinco tablas en el conjunto de datos, y tendrás que usarlas todas para hacer el preprocesamiento de datos y el análisis exploratorio de datos. A continuación se muestra un diccionario de datos que enumera las columnas de cada tabla y describe los datos que contienen.
# 
# - `instacart_orders.csv`: cada fila corresponde a un pedido en la aplicación Instacart.
#     - `'order_id'`: número de ID que identifica de manera única cada pedido.
#     - `'user_id'`: número de ID que identifica de manera única la cuenta de cada cliente.
#     - `'order_number'`: el número de veces que este cliente ha hecho un pedido.
#     - `'order_dow'`: día de la semana en que se hizo el pedido (0 si es domingo).
#     - `'order_hour_of_day'`: hora del día en que se hizo el pedido.
#     - `'days_since_prior_order'`: número de días transcurridos desde que este cliente hizo su pedido anterior.
# - `products.csv`: cada fila corresponde a un producto único que pueden comprar los clientes.
#     - `'product_id'`: número ID que identifica de manera única cada producto.
#     - `'product_name'`: nombre del producto.
#     - `'aisle_id'`: número ID que identifica de manera única cada categoría de pasillo de víveres.
#     - `'department_id'`: número ID que identifica de manera única cada departamento de víveres.
# - `order_products.csv`: cada fila corresponde a un artículo pedido en un pedido.
#     - `'order_id'`: número de ID que identifica de manera única cada pedido.
#     - `'product_id'`: número ID que identifica de manera única cada producto.
#     - `'add_to_cart_order'`: el orden secuencial en el que se añadió cada artículo en el carrito.
#     - `'reordered'`: 0 si el cliente nunca ha pedido este producto antes, 1 si lo ha pedido.
# - `aisles.csv`
#     - `'aisle_id'`: número ID que identifica de manera única cada categoría de pasillo de víveres.
#     - `'aisle'`: nombre del pasillo.
# - `departments.csv`
#     - `'department_id'`: número ID que identifica de manera única cada departamento de víveres.
#     - `'department'`: nombre del departamento.

# # Paso 1. Descripción de los datos
# 

# In[1]:


import pandas as pd
from matplotlib import pyplot as plt
 # importar librerías


# In[2]:


df_instacar_orders=pd.read_csv('/datasets/instacart_orders.csv', sep=';')

df_products=pd.read_csv('/datasets/products.csv', sep=';')

df_aisles=pd.read_csv('/datasets/aisles.csv', sep=';')

df_departments=pd.read_csv('/datasets/departments.csv', sep=';')

df_order_products=pd.read_csv('/datasets/order_products.csv', sep=';')

# leer conjuntos de datos en los DataFrames


# In[3]:


df_instacar_orders.info(show_counts=True) # mostrar información del DataFrame


# In[4]:


df_products.info() # mostrar información del DataFrame


# In[5]:


df_aisles.info()  # mostrar información del DataFrame


# In[6]:


df_departments.info() # mostrar información del DataFrame


# In[7]:


df_order_products.info(show_counts=True) # mostrar información del DataFrame


# # Paso 2. Preprocesamiento de los datos
# 
# Preprocesa los datos de la siguiente manera:
# 
# - Verifica y corrige los tipos de datos (por ejemplo, asegúrate de que las columnas de ID sean números enteros).
# - Identifica y completa los valores ausentes.
# - Identifica y elimina los valores duplicados.
# 
# Asegúrate de explicar qué tipos de valores ausentes y duplicados encontraste, cómo los completaste o eliminaste y por qué usaste esos métodos. ¿Por qué crees que estos valores ausentes y duplicados pueden haber estado presentes en el conjunto de datos?


# In[8]:


print(df_instacar_orders[df_instacar_orders['order_id'].duplicated()].sort_values(by='order_id', ascending=False))
print()
print('El número de pedidos duplicados es:', df_instacar_orders['order_id'].duplicated().sum())
 # Revisa si hay pedidos duplicados


# ¿Tienes líneas duplicadas? Si sí, ¿qué tienen en común?
# Sí. Todas son el mismo día (miércoles) y a la misma hora (2).

# In[9]:


# Basándote en tus hallazgos,
# Verifica todos los pedidos que se hicieron el miércoles a las 2:00 a.m.
df_instacar_orders[(df_instacar_orders['order_dow']==3) & (df_instacar_orders['order_hour_of_day']==2)]


# ¿Qué sugiere este resultado?
# Son 15 de 121 compras que se hicieron ese día a la misma hora, pero representa menos del 1% del total de órdenes.

# In[10]:


# Elimina los pedidos duplicados
df_instacar_orders = df_instacar_orders.drop_duplicates().reset_index(drop=True)


# In[11]:


df_instacar_orders


# In[12]:


# Vuelve a verificar si hay filas duplicadas
df_instacar_orders.duplicated().sum()


# ### `products` data frame

# In[14]:


# Verifica si hay filas totalmente duplicadas
print(df_products.duplicated().sum())


# In[15]:


# Revisa únicamente si hay ID de departamentos duplicados
print(df_products['department_id'].duplicated().sum())


# In[16]:


# Revisa únicamente si hay nombres duplicados de productos (convierte los nombres a letras mayúsculas para compararlos mejor)
df_products['product_name']=df_products['product_name'].str.upper()
productos_nombres_duplicados = df_products['product_name'].duplicated().sum()

print('El número de nombres de productos duplicados es:',productos_nombres_duplicados )


# In[17]:


# Revisa si hay nombres duplicados de productos no faltantes

productos_sin_nombre=df_products['product_name'].isna().sum()
print('La cantidad de productos duplicados sin nombre es:', productos_sin_nombre)

productos_con_nombre= productos_nombres_duplicados-productos_sin_nombre
print('La cantidad de nombres duplicados con nombre es:', productos_con_nombre )

porcentaje_productos_duplicados_sin_nombre=(productos_sin_nombre / productos_nombres_duplicados)*100
print('El porcentaje de productos duplicados sin nombre es de:',porcentaje_productos_duplicados_sin_nombre ,'%')

porcentaje_productos_duplicados_con_nombre=(productos_con_nombre / productos_nombres_duplicados)*100
print('El porcentaje de productos duplicados con nombre es de:', porcentaje_productos_duplicados_con_nombre,'%')


# Describe brevemente tus hallazgos y lo que hiciste con ellos.
# 
# El 99.88% de las compras se realizan en los mismos departamentos.
# El 0.12% de las compras se hacen en departamentos distintos.
# 

# ### `departments` data frame

# In[ ]:


# Revisa si hay filas totalmente duplicadas
print('Hay', df_departments.duplicated().sum(), 'filas duplicadas.')


# In[18]:


# Revisa únicamente si hay IDs duplicadas de productos
print('Hay', df_products['product_id'].duplicated().sum(), 'IDs de productos duplicados.')
print()
print('Cantidad de productos comprados por departamento:', df_products.groupby('department_id')['product_id'].sum())


# Describe brevemente tus hallazgos y lo que hiciste con ellos.
# No estoy seguro de haber entendido esta sección. Las indicaciones son muy confusas.

# ### `aisles` data frame

# In[19]:


# Revisa si hay filas totalmente duplicadas
df_aisles.duplicated().sum()


# In[20]:


# Revisa únicamente si hay IDs duplicadas de productos
print('Hay', df_products['product_id'].duplicated().sum(), 'IDs de productos duplicados.')
print()
print('Cantidad de productos por pasillo:', df_products.groupby('aisle_id')['product_id'].sum())


# Describe brevemente tus hallazgos y lo que hiciste con ellos.
# 
# No estoy seguro de esto. Hay demasiado productos por pasillo.

# ### `order_products` data frame

# In[21]:


# Revisa si hay filas totalmente duplicadas
print('Hay', df_order_products.duplicated().sum(), 'filas duplicadas.')


# In[22]:


# Vuelve a verificar si hay cualquier otro duplicado engañoso
print('Hay', df_order_products['product_id'].duplicated().sum(), 'IDs de productos duplicados.')
print('Hay', df_order_products['order_id'].duplicated().sum(), 'órdenes duplicadas.' )


# In[ ]:


Describe brevemente tus hallazgos y lo que hiciste con ellos.

No hay filas duplicadas, pero hay órdenes y IDs de productos duplicados, lo que indica la cantidad de compras realizadas
y productos comprados.


# ## Encuentra y elimina los valores ausentes
# 
# Al trabajar con valores duplicados, pudimos observar que también nos falta investigar valores ausentes:
# 
# * La columna `'product_name'` de la tabla products.
# * La columna `'days_since_prior_order'` de la tabla orders.
# * La columna `'add_to_cart_order'` de la tabla order_productos.

# ### `products` data frame

# In[24]:


# Encuentra los valores ausentes en la columna 'product_name'
no_name_products=df_products[df_products['product_name'].isna()]
porcentaje_no_name=(no_name_products['product_id'].count()) / (df_products['product_name'].count())*100
print(no_name_products)
print()
print('La cantidad de productos sin  nombre es de:', no_name_products['product_id'].count())
print('El porcentaje de productos sin nombre es de:', porcentaje_no_name,'%')
print('El nombre del pasillo 100 es:', df_aisles.loc[99,'aisle'])
print('El nombre del departamento 21 es:', df_departments.loc[20,'department'])


# Describe brevemente cuáles son tus hallazgos.
# Hay 1258 productos sin nombre, todos relacionados con el pasillo 100 y el departamento con ID=21.

# #  ¿Todos los nombres de productos ausentes están relacionados con el pasillo con ID 100?
# Sí, mismo pasillo (100) y mismo departamento (21)

# Describe brevemente cuáles son tus hallazgos.
# Hay 15 pedidos duplicados en la tabla de Pedidos de Instacar, todos ellos fueron realizados el miércoles a las 2 am.
# 
# En la tabla de Productos hay 49673 id de departamentos duplicados.
# También hay 1361 nombres de productos duplicados, de los cuales 1258 (92.5%) no tienen nombre, y 103 (7.5%) tienen nombre.
# Todos los productos sin nombre están en el pasillo 100 del departamento 21.
# El pasillo 100 no tiene nombre, así como tampoco lo tiene el departamento 21.
# 
# En la tabla Departamentos no hay filas duplicadas, y hay muchos id de productos repetidos por departamento (lo que indica la
# distribución de productos comprados por departamento).

# In[25]:


# ¿Todos los nombres de productos ausentes están relacionados con el departamento con ID 21?
print('Sí')
print()
print(no_name_products.groupby(by='department_id').count())


# Describe brevemente cuáles son tus hallazgos.
# Todos los nombres faltantes se relacionan un pasillo sin nombre. El departamento tampoco tiene nombre.

# In[26]:


# Usa las tablas department y aisle para revisar los datos del pasillo con ID 100 y el departamento con ID 21.
print(df_departments[df_departments['department_id']==21])
print()
print(df_aisles[df_aisles['aisle_id']==100])


# Describe brevemente cuáles son tus hallazgos.
# Ninguno tiene nombre.

# In[27]:


# Completa los nombres de productos ausentes con 'Unknown'
df_products['product_name']=df_products['product_name'].fillna('Unknown')
print(df_products)

print(df_products[df_products['product_name'].isna()].count())


# Describe brevemente tus hallazgos y lo que hiciste con ellos.
# 
# Todos los productos que carecen de nombre están en un pasillo sin nombre, en un departamento que tampoco tiene nombre.
# La cantidad de productos sin nombre representa el 2.5% del total de productos. Debido a que representa un bajo porcentaje,
# y además se desconocen los datos de ubicación, puede prescindirse de ellos.

# ### `orders` data frame

# In[28]:


# Encuentra los valores ausentes
print(df_instacar_orders[df_instacar_orders.isna()])


# In[29]:


# ¿Hay algún valor ausente que no sea el primer pedido del cliente?
print(df_instacar_orders.sort_values('order_number'))


# Describe brevemente tus hallazgos y lo que hiciste con ellos.
# No entiendo bien a qué se refiere esta sección.

# ### `order_products` data frame

# In[30]:


# Encuentra los valores ausentes
print(df_order_products[df_order_products.isna()])


# In[31]:


# ¿Cuáles son los valores mínimos y máximos en esta columna?
print('El valor mínimo es:', df_order_products['add_to_cart_order'].min())
print()
print('El valor máximo es:', df_order_products['add_to_cart_order'].max())


# Describe brevemente cuáles son tus hallazgos.

# In[32]:


# Guarda todas las IDs de pedidos que tengan un valor ausente en 'add_to_cart_order'
missing_add_to_cart_order = df_order_products[df_order_products['add_to_cart_order'].isna()]

order_id_missing = missing_add_to_cart_order['order_id']


# In[33]:


# ¿Todos los pedidos con valores ausentes tienen más de 64 productos?
print('Todos los pedidos con valores ausentes tienen más de 64 productos.')
print()
# Agrupa todos los pedidos con datos ausentes por su ID de pedido.
num_products_noid= df_order_products[df_order_products['order_id'].isin(order_id_missing)][['product_id','order_id']]
num_products_noid= num_products_noid.groupby('order_id').count()
print(num_products_noid.sort_values(by='product_id', ascending=False))
print()
# Cuenta el número de 'product_id' en cada pedido y revisa el valor mínimo del conteo.
print('El valor mínimo del conteo es:', num_products_noid['product_id'].min() )


# Describe brevemente cuáles son tus hallazgos.
# Hay 70 pedidos con valores ausentes, y cada uno tiene una cantidad de productos entre 65 y 127.

# In[34]:


# Remplaza los valores ausentes en la columna 'add_to_cart? con 999 y convierte la columna al tipo entero.
df_order_products['add_to_cart_order']=df_order_products['add_to_cart_order'].fillna(999)
df_order_products['add_to_cart_order']=df_order_products['add_to_cart_order'].astype('int')


# Describe brevemente tus hallazgos y lo que hiciste con ellos.

# ## Conclusiones
# 
# Escribe aquí tus conclusiones intermedias sobre el Paso 2. Preprocesamiento de los datos
# 
# El pasillo 100 y el departamento 21 no tienen nombre, al igual que los productos que están incluidos ahí.


# # Paso 3. Análisis de los datos
# 
# Una vez los datos estén procesados y listos, haz el siguiente análisis:

# # [A] Fácil (deben completarse todos para aprobar)
# 
# 1. Verifica que los valores en las columnas `'order_hour_of_day'` y `'order_dow'` en la tabla orders sean razonables (es decir, `'order_hour_of_day'` oscile entre 0 y 23 y `'order_dow'` oscile entre 0 y 6).
# 2. Crea un gráfico que muestre el número de personas que hacen pedidos dependiendo de la hora del día.
# 3. Crea un gráfico que muestre qué día de la semana la gente hace sus compras.
# 4. Crea un gráfico que muestre el tiempo que la gente espera hasta hacer su siguiente pedido, y comenta sobre los valores mínimos y máximos.

# ### [A1] Verifica que los valores sean sensibles

# In[35]:


print(df_instacar_orders.info())
print()
print(df_instacar_orders['order_hour_of_day'].min(), df_instacar_orders['order_hour_of_day'].max() )


# In[36]:


print(df_instacar_orders['order_dow'].min(), df_instacar_orders['order_dow'].max() )


# Escribe aquí tus conclusiones
# Ambas columnas cumplen con las condiciones.

# ### [A2] Para cada hora del día, ¿cuántas personas hacen órdenes?

# In[37]:


persons_orders_per_hour = df_instacar_orders.groupby('order_hour_of_day')['order_id'].count()

print(persons_orders_per_hour)

persons_orders_per_hour.plot(x='order_hour_of_day', y='order_id', title='Cantidad de personas que hacen compras por hora del día',
                      kind='bar', xlabel='Hora del día', ylabel='Cantidad de personas',figsize=(10,10))

plt.show()




# Escribe aquí tus conclusiones
# 
# La mayoría de la gente prefiere comprar entre las 9 de la mañana y las 5 de la tarde (9 a 17).
# El horario donde hay menor cantidad de gente comprando es entre la 1 y las 5 de la mañana.

# ### [A3] ¿Qué día de la semana compran víveres las personas?

# In[38]:


persons_orders_per_day = df_instacar_orders.groupby('order_dow')['order_id'].count()

print(persons_orders_per_day)

persons_orders_per_day.plot(x='order_dow', y='order_id', title='Cantidad de personas que hacen compras por día de la semana',
                      kind='bar', xlabel='Día de la semana', ylabel='Cantidad de personas',figsize=(10,10))
plt.show()


# Escribe aquí tus conclusiones
# La gente prefiere comprar los días domingo y lunes, en el resto de la semana no hay mucha variación.

# ### [A4] ¿Cuánto tiempo esperan las personas hasta hacer otro pedido? Comenta sobre los valores mínimos y máximos.

# In[39]:


waiting_days = df_instacar_orders.groupby('days_since_prior_order')['user_id'].count()
print(waiting_days.sort_values(ascending= True))


# Escribe aquí tus conclusiones
# 
# Hay 51,337 personas (el valor máximo) que esperan 30 días para hacer la siguiente compra, mientras que hay 2640 personas
# (el valor mínimo) que esperan 26 días para hacer la siguiente compra.
# 

# # [B] Intermedio (deben completarse todos para aprobar)
# 
# 1. ¿Existe alguna diferencia entre las distribuciones `'order_hour_of_day'` de los miércoles y los sábados? Traza gráficos de barra de `'order_hour_of_day'` para ambos días en la misma figura y describe las diferencias que observes.
# 2. Grafica la distribución para el número de órdenes que hacen los clientes (es decir, cuántos clientes hicieron solo 1 pedido, cuántos hicieron 2, cuántos 3, y así sucesivamente...).
# 3. ¿Cuáles son los 20 principales productos que se piden con más frecuencia (muestra su identificación y nombre)?

# ### [B1] Diferencia entre miércoles y sábados para  `'order_hour_of_day'`. Traza gráficos de barra para los dos días y describe las diferencias que veas.

# In[40]:


wednesday_orders = df_instacar_orders[df_instacar_orders['order_dow']==3]
wednesday_orders = wednesday_orders.groupby('order_hour_of_day')['order_id'].count()
print(wednesday_orders.sort_values(ascending= True))
print()


# In[41]:


wednesday_orders.plot(x='order_hour_of_day' , y='order_id' , title='Cantidad de compras por hora en miércoles',
                      kind='bar', xlabel='Hora del día', ylabel='Cantidad de compras',figsize=(10,10))
plt.show()


# In[42]:


saturday_orders = df_instacar_orders[df_instacar_orders['order_dow']==6]
saturday_orders = saturday_orders.groupby('order_hour_of_day')['order_id'].count()
print(saturday_orders.sort_values(ascending=True))


# In[43]:


saturday_orders.plot(x='order_hour_of_day' , y='order_id' , title='Cantidad de compras por hora en sábado',
                      kind='bar', xlabel='Hora del día', ylabel='Cantidad de compras',figsize=(10,10))
plt.show()


# #### Escribe aquí tus conclusiones
# 
# Miércoles: 
#     mayor - de 9am a 4pm
#     menor - de 0am a 6am
# Sábados: 
#     mayor - de 10am a 4pm
#     menor - de 0am a 6am
# 
# En ambos días tienen patrones de compra muy semejantes. El rango de horario con más tráfico es muy parecido en ambos días,
# así como el horario con menos tráfico.

# ### [B2] ¿Cuál es la distribución para el número de pedidos por cliente?

# In[44]:


orders_distribution = df_instacar_orders.groupby(by='order_number')['user_id'].count()
print(orders_distribution.head(10))
print()
print(orders_distribution.tail(10))
mean_orders=df_instacar_orders.groupby(by='user_id')['order_number'].sum().median()
print()
print(mean_orders)


# In[45]:


orders_distribution.plot(x='order_number' , y='user_id' , title='Distribución de pedidos por cliente',
                      kind='bar', xlabel='Veces que se ha realizado una orden', ylabel='Cantidad de usuarios',figsize=(25,10))
plt.show()


# Escribe aquí tus conclusiones
# 
# La mayoría de la gente ha comprado en promedio 13 veces.

# ### [B3] ¿Cuáles son los 20 productos más populares (muestra su ID y nombre)?

# In[46]:


reordered_list = df_order_products[df_order_products['reordered']==1]

reordered_products = df_products.merge(reordered_list, on='product_id')

most_wanted = df_instacar_orders.merge(reordered_products, on='order_id')


# In[47]:


most_wanted_names= most_wanted.groupby('product_name')['product_id'].count().sort_values(ascending=False)

print(most_wanted_names.head(20))


# In[48]:


most_wanted_id=most_wanted.groupby('product_id')['product_id'].count().sort_values(ascending=False)

print(most_wanted_id.head(20))


# In[ ]:





# Escribe aquí tus conclusiones
# 
# Los 20 productos más comprados son frutas.

# # [C] Difícil (deben completarse todos para aprobar)
# 
# 1. ¿Cuántos artículos suelen comprar las personas en un pedido? ¿Cómo es la distribución?
# 2. ¿Cuáles son los 20 principales artículos que vuelven a pedirse con mayor frecuencia (muestra sus nombres e IDs de los productos)?
# 3. Para cada producto, ¿cuál es la tasa de repetición del pedido (número de repeticiones de pedido/total de pedidos?
# 4. Para cada cliente, ¿qué proporción de los productos que pidió ya los había pedido? Calcula la tasa de repetición de pedido para cada usuario en lugar de para cada producto.
# 5. ¿Cuáles son los 20 principales artículos que la gente pone primero en sus carritos (muestra las IDs de los productos, sus nombres, y el número de veces en que fueron el primer artículo en añadirse al carrito)?

# ### [C1] ¿Cuántos artículos compran normalmente las personas en un pedido? ¿Cómo es la distribución?

# In[49]:


complete_df = df_instacar_orders.merge(reordered_products, on='order_id')
mean_products_per_person = complete_df.groupby('order_id')['product_id'].count()


# In[50]:


print(mean_products_per_person.sort_values())
print()
print(mean_products_per_person.mean())
print()
mean_products_per_person.plot(kind='hist', bins=100, figsize=(20,20) )
plt.show()


# In[51]:


print('Las personas compran, en promedio: ',mean_products_per_person.mean(),' artículos en un pedido.')


# Escribe aquí tus conclusiones

# ### [C2] ¿Cuáles son los 20 principales artículos que vuelven a pedirse con mayor frecuencia (muestra sus nombres e IDs de los productos)?

# In[52]:


veinte_nombres_principales= complete_df.groupby(by='product_name').count()
veinte_principales = veinte_nombres_principales.sort_values(by='order_id',ascending=False)


# In[53]:


print(veinte_principales[['reordered']].head(20))


# In[54]:


veinte_IDs_principales= complete_df.groupby(by='product_id').count()
veinte_IDs = veinte_IDs_principales.sort_values(by='order_id',ascending=False)


# In[55]:


print(veinte_IDs[['reordered']].head(20))


# Escribe aquí tus conclusiones
# 
# Todos los productos más comprados son frutas y verduras, predominando los productos orgánicos.

# ### [C3] Para cada producto, ¿cuál es la proporción de las veces que se pide y que se vuelve a pedir?

# In[56]:


print(df_order_products.groupby('product_id')['reordered'].sum())
print()
print(df_order_products.groupby('product_id')['product_id'].count())


# In[57]:


agg_orders={'reordered':'sum', 'product_id':'count'}
agg_mean=df_order_products.groupby('product_id').agg(agg_orders)


# In[58]:


agg_mean['promedio_de_recompra'] = agg_mean['reordered'] / agg_mean['product_id']


# In[59]:


agg_mean


# Escribe aquí tus conclusiones

# ### [C4] Para cada cliente, ¿qué proporción de sus productos ya los había pedido?


# In[60]:


compras_totales = df_instacar_orders.merge(df_order_products, on='order_id')

compras_totales


# In[61]:


#Cuánto compra cada user_id
print(compras_totales.groupby('user_id')['reordered'].mean())


# Escribe aquí tus conclusiones


# ### [C5] ¿Cuáles son los 20 principales artículos que las personas ponen primero en sus carritos?

# In[62]:


repetidos = complete_df[complete_df['add_to_cart_order']==1]


# In[63]:


veinte_primeros = repetidos.groupby(by='product_name').count()
veinte_primeros = veinte_primeros.sort_values(by='add_to_cart_order', ascending=False)


# In[64]:


print(veinte_primeros.head(20))


# Escribe aquí tus conclusiones
# Los veinte productos que los clientes ponen primero en su carrito, son en su mayoría frutas, pero también incluye bebidas,
# como soda, agua mineral y leche.

# ### Conclusion general del proyecto:

# La gente prefiere comprar durante la mañana.
# La mayoría de la gente compra frutas y bebidas.
# Los productos más solicitados son frutas.
# Las personas compran, en promedio, 6 artículos por pedido.
# La gente compra, en promedio, 13 veces.
# La mayoría de la gente espera hasta 30 días para volver a realizar un pedido.
# 
# Nota: Me ha resultado sumamente difícil identificar qué es lo que se requiere a lo largo del proyecto. La redacción me parece
#     muy poco clara, y el objetivo demasiado ambiguo. Confieso que no sé si lo que he hecho, sea correcto y lo que se espera.
#     Esta situación la he comentado desde el sprint 2. Lo que hace que el material, que ya de por sí es complicado, lo sea aún
#     más.

