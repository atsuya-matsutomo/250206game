package main

import (
	"math/rand"

	"github.com/hajimehoshi/ebiten/v2"
	"github.com/hajimehoshi/ebiten/v2/inpututil"
)

type lifeGame struct {
	isUpdate bool   // 更新するか
	frame    int    // フレーム数
	fps      int    // fps
	cells    []cell // セルの状態
	width    int    // セルの横幅
	height   int    // セルの縦幅

	mousePos        vec2 // マウスの位置
	isMousePressing bool // マウスが押されているか
	filledPos       vec2 // マウスで塗りつぶした位置
}

func newLifeGame(fps, width, height, fillRate int) *lifeGame {
	return &lifeGame{
		isUpdate: true,
		frame:    0,
		fps:      fps,
		cells:    initializeCells(width, height),
		width:    width,
		height:   height,
	}
}

func initializeCells(width, height int) []cell {
	cells := make([]cell, width*height)
	for i := 0; i < len(cells); i++ {
		cellX := i % width
		cellY := i / width
		cellIsAlive := rand.Intn(100) < fillRate
		cell := newCell(cellX, cellY, cellIsAlive)
		cells[i] = *cell
	}
	return cells
}

func (life *lifeGame) draw(pixels []byte) {
	for i := 0; i < len(life.cells); i++ {
		life.cells[i].draw(life.width, life.height, pixels)
	}
}

func (game *lifeGame) updateMouse() {
	game.mousePos.x, game.mousePos.y = ebiten.CursorPosition()
	if ebiten.IsMouseButtonPressed(ebiten.MouseButtonLeft) {
		game.isMousePressing = true
	}

	if inpututil.IsMouseButtonJustReleased(ebiten.MouseButtonLeft) {
		game.isMousePressing = false
	}

	if game.isMousePressing {
		if game.mousePos.x != game.filledPos.x || game.mousePos.y != game.filledPos.y {
			cellX := game.mousePos.x
			cellY := game.mousePos.y

			if cellX < 0 || cellY < 0 || game.width <= cellX || game.height <= cellY {
				return
			}

			cell := &game.cells[cellY*game.width+cellX]
			isAlive := !cell.isAlive
			cell.setIsAlive(isAlive)
			cell.setIsAlive(isAlive) // 2回呼び出すことで、恒久的な色を反映
			game.filledPos = game.mousePos
		}
	}
}

func (game *lifeGame) update() {
	// リセット処理
	if inpututil.IsKeyJustPressed(ebiten.KeyR) {
		game.cells = initializeCells(game.width, game.height)
	}

	// ポーズ処理
	if inpututil.IsKeyJustPressed(ebiten.KeySpace) {
		game.isUpdate = !game.isUpdate
		game.frame = 0
	}

	game.updateMouse()

	if !game.isUpdate {
		return
	}

	game.frame++
	if game.frame%(60/game.fps) != 0 {
		return
	}

	nextAlives := make([]bool, len(game.cells))

	for y := 0; y < game.height; y++ {
		for x := 0; x < game.width; x++ {
			cell := game.cells[y*game.width+x]
			pop := game.neighbourCount(x, y)
			switch {
			case pop < 2:
				nextAlives[y*game.width+x] = false
			case (pop == 2 || pop == 3) && cell.isAlive:
				nextAlives[y*game.width+x] = true
			case pop > 3:
				nextAlives[y*game.width+x] = false
			case pop == 3:
				nextAlives[y*game.width+x] = true
			}
		}
	}

	for i := 0; i < len(game.cells); i++ {
		game.cells[i].setIsAlive(nextAlives[i])
	}
}

func (game *lifeGame) neighbourCount(x, y int) int {
	count := 0
	for difY := -1; difY <= 1; difY++ {
		for difX := -1; difX <= 1; difX++ {
			if difX == 0 && difY == 0 {
				continue
			}
			nx := x + difX
			ny := y + difY
			if nx < 0 || ny < 0 || game.width <= nx || game.height <= ny {
				continue
			}
			if game.cells[ny*game.width+nx].isAlive {
				count++
			}
		}
	}
	return count
}
