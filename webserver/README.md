#### Example how to make container image to test apache server


How to build and release image :

$docker build -t [name of container] .

- if dockerfile exists in another route then use that route in place of '.'

$docker login
$docker push [name of container] # release that image

