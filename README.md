#### Este proyecto consistió en desarrollar una API de dónde se pudieran realizar consultas con las siguientes funciones(ver al final)
La API se puede consultar con deta space en el siguiente link: 
https://deta.space/discovery/r/g25mfivm4mnthufp

Como primer paso se limpiaron los datos de las bases proporcionadas. 
EL archivo limpieza_datos contiene el notebook donde se trabajó. 

El segundo proyecto se trato de realizar un sistema de recomendación de películas. 
Todo el proceso esta documentado en el notebook 
@app.get("/max-duration")
async def get_max_duration(year: int = None, platform: str = None, duration_type: str = None):
    
    """ 
        Dados los parámetros opcionales año, plataforma y tipo de duración, esta función carga un archivo CSV que contiene 
        información sobre películas y programas de TV, filtra los datos según los parámetros y devuelve una tupla que 
        contiene el valor máximo de duración, el tipo de duración y el título de la película con la duración máxima.

    Args:
    
        year (int, opcional): Si se especifica, filtra los datos para incluir solo películas y programas de TV lanzados 
        en ese año. Los valores para year van de 1920 a 2021 (inclusivo).
        
        platform (str, opcional): Si se especifica, filtra los datos para incluir solo películas y programas de TV 
        disponibles en una plataforma de streaming (Amazon, Disney, Hulu, Netflix) que comience con la letra especificada.
        De no especificarse, devolverá la película o serie de mayor duración para todas las plataformas.
        
        duration_type (str, opcional): Si se especifica, filtra los datos para incluir solo películas o programas de TV 
        con este tipo de duración (por ejemplo, 'min' para películas, 'season' para programa de TV).

    Returns:
    
        Una tupla que contiene tres valores:
        - max_duration (int): El valor máximo de la columna 'duration_int' en los datos filtrados.
        - max_duration_type (str): El valor de la columna 'duration_type' para la película o programa de TV con la 
          duración máxima.
        - max_duration_title (str): El valor de la columna 'title' para la película o programa de TV con la duración 
          máxima.
    """
    
    # Crea data frame 
    df = pd.read_csv(StringIO(content_plata))

    # Filtra el data frame con los parámetros opcionales
    if year is not None:
        
         # Verifica si el año es válido
        if year in range(1920,2022):
        
            # Filtra por año
            df = df[df['release_year'] == year]
            
        else: 
            return "Year sólo toma valores en el rango 1920 - 2021"
        
    if platform is not None:
        
        # Convierte la plataforma a minúsculas
        platform = platform.lower()
    
        # Verifica si la plataforma es válida
        if 'net' in platform: 
            platform = 'n'
        elif 'hu' in platform:
            platform = 'h'
        elif 'di' in platform: 
            platform = 'd'
        elif 'am' in platform:
            platform = 'a'
        else:
            # Devuelve un mensaje de error si la plataforma no es válida
            return "Platform sólo toma alguna de estas opciones: 'amazon', 'disney', 'hulu', 'netflix'"
            
        # Filtra el DataFrame por la plataforma especificada
        df = df[df['show_id'].str[0] == platform]
        
        
    if duration_type is not None:
        
        # Verifica si el tipo es válido
        if duration_type in ('min', 'season'):
            
            # Filtra por tipo
            df = df[df['duration_type'] == duration_type]
            
        else:
            return "duration_type incorrecto. Escriba 'min' para películas o 'season' para series"

    # Obtine el máximo valor de duration_int y el título de la película
    max_duration = df['duration_int'].max()
    max_duration_type = df.loc[df['duration_int'].idxmax(), 'duration_type']
    max_duration_title = df.loc[df['duration_int'].idxmax(), 'title']

    # Close the file
    plata.close()

    # Return the results
    return {
        'max_duration': max_duration,
        'max_duration_type': max_duration_type,
        'max_duration_title': max_duration_title
    }

@app.get("/score-count/")
async def get_score_count(score: float, platform: str = None, year: int = None):
    
    """ 
       Regresa el número de películas con una calificación mayor a cierto puntaje (score) 
       para los principales servicios de streaming (amazon, disney, hulu, netflix). 
       
       Filtra por servicio (platform=amazon, platform=disney, platform=hulu, platform=netflix)
       si se utiliza el parámetro opcional 'platform'.
       
       Filtra por año de lanazamiento (con valores en un rango de 1920 - 2021 inclusivo) 
       si se utiliza el parámetro opcional 'year'. 

    Args:
    
        platform (_str_): una de las siguintes: amazon, disney, hulu, netflix 
        
        score (_float_): valor entre 3.33 y 3.72
        
        year (_int_): valor entre 1920 y 2021

    Returns:
    
        num_movies (_int_): número de películas con puntaje mayor al especificado. 
    """
    # Carga el archivo CSV como un DataFrame
    df = pd.read_csv(StringIO(content_plata))
    
    # Verifica que se haya asignado valor al parámetro score y que sea menor al máximo
    if score == None:
        return {"error": "Debe asignar al parámetro 'score' un valor entre 3.33 y 3.72"}
    
    elif score > 3.72: 
        return {"error": "Debe asignar al parámetro 'score' un valor menor a 3.72"}
    
    # En caso de que al parámetro opcional platform se le haya asignado valor
    if platform != None: 
        
        # Convierte la plataforma a minúsculas
        platform = platform.lower()
    
        # Verifica si la plataforma es válida
        if 'net' in platform: 
            platform = 'n'
        elif 'hu' in platform:
            platform = 'h'
        elif 'di' in platform: 
            platform = 'd'
        elif 'am' in platform:
            platform = 'a'
        else:
            # Devuelve un mensaje de error si la plataforma no es válida
            return {"error": "Platform sólo toma alguna de estas opciones: 'amazon', 'disney', 'hulu', 'netflix'"}
            
        # Filtra el DataFrame por la plataforma especificada
        df = df[df['show_id'].str[0] == platform]
    
    # En caso de que al parámetro opcional year se le haya asignado valor
    if year != None:
        
        # Verifica si el año es válido
        if year in range(1920,2022):
        
            # Cuenta el número de películas que cumplen con los criterios de año y score
            num_movies = ((df['release_year'] == year) & (df['scores'] > score)).sum()
            
        else: 
            return {"error": "Year sólo toma valores en el rango 1920 - 2021"}
    
    # Si no se seleccionaron parámetros opcionales    
    else: 
        
        # Cuenta el número de películas que cumplen con el criterio de score
        num_movies = int((df['scores'] > score).sum())
        
    return {'num_movies': num_movies}
    

@app.get("/count-platform/")  
async def get_count_platform(platform: str):
    """
    Cuenta el número de películas en una plataforma de streaming específica 
    basándose en un archivo CSV que contiene una columna 'show_id' que indica 
    la plataforma de cada película.

    Args:
    
        platform (str): La abreviatura de la plataforma de streaming que se desea contar. 
                        Puede ser una de las siguientes opciones: 'net' para Netflix, 
                        'hu' para Hulu, 'di' para Disney, o 'am' para Amazon.

    Returns:
    
        int: El número de películas en la plataforma de streaming especificada.

        str: Si la plataforma especificada no es reconocida, se devuelve un mensaje de error.

    """

    # Carga el archivo CSV como un DataFrame
    df = pd.read_csv(StringIO(content_plata))

    # Convierte la plataforma a minúsculas
    platform = platform.lower()

    # Verifica si la plataforma es válida
    if 'net' in platform: 
        platform = 'n'
    elif 'hu' in platform:
        platform = 'h'
    elif 'di' in platform: 
        platform = 'd'
    elif 'am' in platform:
        platform = 'a'
    else:
        # Devuelve un mensaje de error si la plataforma no es válida
        return {"error": "Platform sólo toma alguna de estas opciones: 'amazon', 'disney', 'hulu', 'netflix'"}

    # Filtra el DataFrame para incluir solo las películas de la plataforma seleccionada
    num_pelis = int((df['show_id'].str[0] == platform).sum())

    return {"num_pelis":num_pelis}


@app.get("/actor/")
async def get_actor(platform: str = None, year: int = None):
    
    """
    Obtiene el actor con más menciones de una plataforma y un año específicos.

    Args:
    
        platform (str): El nombre de la plataforma en minúsculas (ej: 'netflix','disney', 'amazon').
        
        year (int): El año de lanzamiento de la película o serie a buscar.

    Returns:
    
        str: El nombre del actor con más menciones en la plataforma y año especificados.
        
        int: El número de veces que el actor aparece en distintas películas

        Si la plataforma ingresada no es válida (no se encuentra en la lista de plataformas soportadas), se devuelve
        una cadena que indica que el nombre de la plataforma es incorrecto.

    """
    # Carga el archivo CSV como un DataFrame
    df = pd.read_csv(StringIO(content_plata))
    
    # En caso de que al parámetro opcional platform se le haya asignado valor
    if platform != None: 
        
        # Convierte la plataforma a minúsculas
        platform = platform.lower()
    
        # Verifica si la plataforma es válida
        if 'net' in platform: 
            platform = 'n'
        elif 'di' in platform: 
            platform = 'd'
        elif 'hu' in platform:
            return {"disculpas": "no se tiene información sobre actor para plataforma 'hulu'"}
        elif 'am' in platform:
            platform = 'a'
        else:
            # Devuelve un mensaje de error si la plataforma no es válida
            return {"error": "Platform sólo toma alguna de estas opciones: 'amazon', 'disney', 'netflix'"}
            
        # Filtra el DataFrame por la plataforma especificada
        df = df[df['show_id'].str[0] == platform]
    
    # En caso de que al parámetro opcional year se le haya asignado valor
    if year != None:
        
        # Verifica si el año es válido
        if year in range(1920,2022):
        
            # Filtra el Data Frame por año especificado
            df = df[df['release_year'] == year]
            
        else: 
            return {"error": "Year sólo toma valores en el rango 1920 - 2021"}
    
    # Concatena las cadenas en la columna 'cast'
    cast_string = df['cast'].str.cat(sep=',')

    # Separa la cadena concatenada por comas para obtener una lista de todos los actores
    cast_list = cast_string.split(',')

    # Crea un diccionario para contar el número de apariciones de cada actor
    actor_counts = {}
    for actor in cast_list:
        if actor in actor_counts:
            actor_counts[actor] += 1
        else:
            actor_counts[actor] = 1

    # Encuentra el actor con más apariciones
    most_common_actor = max(actor_counts, key=actor_counts.get)

    return {"most_common_actor":most_common_actor, "num_apariciones":actor_counts[most_common_actor]}
