constrain layout - wrap_content will always allow all items to be displayed, while 0dp will take available space

If you have some view+ recycler view, and both have to be scrollable then view needs to be a header/footer of recycler view
recycler view can have cache for viewHolder before they are needed to use to provide smooth experience
RecyclerViewyou can setup layoutManager inside xml
wszystko powinno byc enkapsulowane - viewHolder nie ma referencji bezposrednio do ViewHoldera

StackView allows displaying view as stacked with animation to remove them

View has a fun animate() which allows to animate it

You can pass ResId directly to view.text

Androix Webkit suppors dark mode

ViewPager2 allows vertical scroll, and is easier to use

NAVIGATION:
Navigation listeners - allows to control locking navigation view


ANIMATIONS:
Exit Transition - played for UIcontroller that will be taken off from the screen
Enter Transition - played for UIController that will be displayed on the screen
pop Exit Transition - played for UiController that will be takaen off from the screen because of back button click
pop Enter Transition - played for UIController that will be displayd on the screen because of back button click