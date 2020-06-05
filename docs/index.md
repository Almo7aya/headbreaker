<script src="https://cdnjs.cloudflare.com/ajax/libs/konva/6.0.0/konva.js" integrity="sha256-F/REgXgQ84YI/OLq+RUNixRhAxFw/ShOx2A/cEpi3QA=" crossorigin="anonymous"></script>
<script src="../js/headbreaker.js"></script>

<div id="container">

</div>

<script>
    function commitAnchors(model, group) {
        model.payload.x = group.x();
        model.payload.y = group.y();
        console.log(['drag', model.payload.id, model.payload.x, model.payload.y, group.x(), group.y()]);
    }

    function anchorsDelta(model, group) {
        return [
        group.x() - model.payload.x,
        group.y() - model.payload.y
        ];
    }

    function createPoint(insert, t, s, n) {
        return insert.isTab() ? t : insert.isSlot() ? s : n;
    }

    function createPoints(model, size = 50) {
        return [
            0, 0,
            1, 0,
            2, createPoint(model.up, -1, 1, 0),
            3, 0,
            4, 0,
            4, 1,
            createPoint(model.right, 5, 3, 4), 2,
            4, 3,
            4, 4,
            3, 4,
            2, createPoint(model.down, 5, 3, 4),
            1, 4,
            0, 4,
            0, 3,
            createPoint(model.left, -1, 1, 0), 2,
            0, 1
        ].map(it => it * size / 5)
    }

    function renderPiece(layer, model) {
        var group = new Konva.Group({
        x: model.centralAnchor.x,
        y: model.centralAnchor.y
        });

        var piece = new Konva.Line({
        points: createPoints(model),
        fill: (model.payload.color || '#00D2FF'),
        stroke: 'black',
        strokeWidth: 3,
        closed: true,
        });
        group.add(piece);
        layer.add(group);
        group.draggable('true')

        group.on('mouseover', function () {
        document.body.style.cursor = 'pointer';
        });
        group.on('mouseout', function () {
        document.body.style.cursor = 'default';
        });

        commitAnchors(model, group);

        group.on('dragmove', function () {
        let [dx, dy] = anchorsDelta(model, group);

        if (!headbreaker.isNullVector(dx, dy)) {
            model.drag(dx, dy, true)
            commitAnchors(model, group);
            layer.draw();
        }
        });

        group.on('dragend', function () {
        model.drop();
        layer.draw();
        })

        model.onTranslate((dx, dy) => {
        group.x(model.centralAnchor.x)
        group.y(model.centralAnchor.y)
        commitAnchors(model, group);
        })
    }

    var stage = new Konva.Stage({
        container: 'container',
        width: 900,
        height: 900
    });

    var layer = new Konva.Layer();
    stage.add(layer);

    const puzzle = new headbreaker.Puzzle(25, 10);

    function createPiece(layer, puzzle, config, x, y, payload) {
        let piece = puzzle.newPiece(config);
        piece.payload = payload;
        piece.placeAt(headbreaker.anchor(x, y));
        renderPiece(layer, piece);
    }

    createPiece(layer, puzzle,
        {up: headbreaker.None, right: headbreaker.Tab, down: headbreaker.Tab, left: headbreaker.Slot},
        0, 0,
        {id: 'a', color: 'red'});
    createPiece(layer, puzzle,
        {up: headbreaker.Slot, right: headbreaker.Tab, down: headbreaker.Tab, left: headbreaker.Slot},
        50, 0,
        {id: 'b'});
    createPiece(layer, puzzle,
        {up: headbreaker.Slot, right: headbreaker.Tab, down: headbreaker.Tab, left: headbreaker.Slot},
        100, 0,
        {id: 'c'});
    createPiece(layer, puzzle,
        {up: headbreaker.Slot, right: headbreaker.None, down: headbreaker.Slot, left: headbreaker.Slot},
        100, 50,
        {id: 'd'});


    createPiece(layer, puzzle,
        {up: headbreaker.Slot, right: headbreaker.Slot, down: headbreaker.Slot, left: headbreaker.Slot},
        200, 150,
        {id: 'e', color: 'green'});
    createPiece(layer, puzzle,
        {up: headbreaker.Tab, right: headbreaker.Tab, down: headbreaker.Tab, left: headbreaker.Tab},
        300, 200,
        {id: 'f', color: 'purple'});

    puzzle.autoconnectAll();
    layer.draw()

</script>