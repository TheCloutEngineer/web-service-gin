# Write a handler to add a new item

When the client makes a POST request at /albums, you want to add the album described in the request body to the existing albums’ data.

To do this, you’ll write the following:

Logic to add the new album to the existing list.
A bit of code to route the POST request to your logic.
Write the code¶

Add code to add albums data to the list of albums.

Somewhere after the import statements, paste the following code. (The end of the file is a good place for this code, but Go doesn’t enforce the order in which you declare functions.)
```go
// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}
```

In this code, you:

Use Context.BindJSON to bind the request body to newAlbum.
Append the album struct initialized from the JSON to the albums slice.
Add a 201 status code to the response, along with JSON representing the album you added.
Change your main function so that it includes the router.POST function, as in the following.

func main() {
router := gin.Default()
router.GET("/albums", getAlbums)
router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
In this code, you:

Associate the POST method at the /albums path with the postAlbums function.

With Gin, you can associate a handler with an HTTP method-and-path combination. In this way, you can separately route requests sent to a single path based on the method the client is using.

Run the code¶

If the server is still running from the last section, stop it.

From the command line in the directory containing main.go, run the code.

$ go run .
From a different command line window, use curl to make a request to your running web service.

$ curl http://localhost:8080/albums \
--include \
--header "Content-Type: application/json" \
--request "POST" \
--data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
The command should display headers and JSON for the added album.

HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Wed, 02 Jun 2021 00:34:12 GMT
Content-Length: 116

{
"id": "4",
"title": "The Modern Sound of Betty Carter",
"artist": "Betty Carter",
"price": 49.99
}
As in the previous section, use curl to retrieve the full list of albums, which you can use to confirm that the new album was added.

$ curl http://localhost:8080/albums \
--header "Content-Type: application/json" \
--request "GET"
The command should display the album list.

[
{
"id": "1",
"title": "Blue Train",
"artist": "John Coltrane",
"price": 56.99
},
{
"id": "2",
"title": "Jeru",
"artist": "Gerry Mulligan",
"price": 17.99
},
{
"id": "3",
"title": "Sarah Vaughan and Clifford Brown",
"artist": "Sarah Vaughan",
"price": 39.99
},
{
"id": "4",
"title": "The Modern Sound of Betty Carter",
"artist": "Betty Carter",
"price": 49.99
}
]
In the next section, you’ll add code to handle a GET for a specific item.

Write a handler to return a specific item¶

When the client makes a request to GET /albums/[id], you want to return the album whose ID matches the id path parameter.

To do this, you will:

Add logic to retrieve the requested album.
Map the path to the logic.
Write the code¶

Beneath the postAlbums function you added in the preceding section, paste the following code to retrieve a specific album.

This getAlbumByID function will extract the ID in the request path, then locate an album that matches.

// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
id := c.Param("id")

    // Loop over the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
In this code, you:

Use Context.Param to retrieve the id path parameter from the URL. When you map this handler to a path, you’ll include a placeholder for the parameter in the path.

Loop over the album structs in the slice, looking for one whose ID field value matches the id parameter value. If it’s found, you serialize that album struct to JSON and return it as a response with a 200 OK HTTP code.

As mentioned above, a real-world service would likely use a database query to perform this lookup.

Return an HTTP 404 error with http.StatusNotFound if the album isn’t found.

Finally, change your main so that it includes a new call to router.GET, where the path is now /albums/:id, as shown in the following example.

func main() {
router := gin.Default()
router.GET("/albums", getAlbums)
router.GET("/albums/:id", getAlbumByID)
router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}
In this code, you:

Associate the /albums/:id path with the getAlbumByID function. In Gin, the colon preceding an item in the path signifies that the item is a path parameter.
Run the code¶

If the server is still running from the last section, stop it.

From the command line in the directory containing main.go, run the code to start the server.

$ go run .
From a different command line window, use curl to make a request to your running web service.

$ curl http://localhost:8080/albums/2
The command should display JSON for the album whose ID you used. If the album wasn’t found, you’ll get JSON with an error message.

{
"id": "2",
"title": "Jeru",
"artist": "Gerry Mulligan",
"price": 17.99
}
Conclusion¶

Congratulations! You’ve just used Go and Gin to write a simple RESTful web service.

Suggested next topics:

If you’re new to Go, you’ll find useful best practices described in Effective Go and How to write Go code.
The Go Tour is a great step-by-step introduction to Go fundamentals.
For more about Gin, see the Gin Web Framework package documentation or the Gin Web Framework docs.
Completed code¶

This section contains the code for the application you build with this tutorial.

package main

import (
"net/http"

    "github.com/gin-gonic/gin"
)

// album represents data about a record album.
type album struct {
ID     string  `json:"id"`
Title  string  `json:"title"`
Artist string  `json:"artist"`
Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

func main() {
router := gin.Default()
router.GET("/albums", getAlbums)
router.GET("/albums/:id", getAlbumByID)
router.POST("/albums", postAlbums)

    router.Run("localhost:8080")
}

// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
c.IndentedJSON(http.StatusOK, albums)
}

// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}

// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
id := c.Param("id")

    // Loop through the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}